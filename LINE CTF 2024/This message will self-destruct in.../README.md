# This message will self-destruct in...

![Untitled](This%20message%20will%20self-destruct%20in%20957cb5ba381a47f5a40fd42224c9d8fa/Untitled.png)

| framework | flask |
| --- | --- |
| vulnerability | race condition(infinite redirection) & logical bug |

# Description

This website enables users to upload an image file along with a password and generates a mosaicked version of the image. It then provides the user with a link to access the mosaicked image.

When a user navigates to the provided URL, a timeout is set to 10 seconds. During this time, if the user enters the correct password, the server returns the original uploaded image.

And there is a way to generate mosaicked flag image, but there are no way to bypass password and access to original image.

# Code Analysis

If we upload the image server call `__add_image()` function.

```python
def __add_image(password, id_, file=None, image_url=None, admin=False):
    t = Thread(target=convert_and_save, args=(id_, file, image_url))
    t.start()

    # no need, but time to waiting heavy response makes me excited!!
    if not admin:
        time.sleep(5)

    if file:
        mimetype = file.content_type
    elif image_url.endswith('.jpg'):
        mimetype = 'image/jpg'
    else:
        mimetype = 'image/png'

    db.add_image(id_, mimetype, password)

    return urljoin(URLBASE, id_)
```

The __add_image() function generate thread which generate mosaicked image, and save the mosaicked, and original image to the image directory.

The fun fact is on the main Thread there are no verification of the given file or url is image or not.

```python
def convert_and_save(id, file=None, url=None):
    try:
        if url:
            res = requests.get(url, timeout=3)
            image_bytes = res.content
        elif file:
            image_bytes = io.BytesIO()
            file.save(image_bytes)
            image_bytes = image_bytes.getvalue()

        if len(image_bytes) > app.config['MAX_CONTENT_LENGTH']:
            raise Exception('image too large')

        obfs_image_bytes = util.mosaic(image_bytes)

        with open(os.path.join(FILE_SAVE_PATH, id), 'wb') as f:
            f.write(image_bytes)
        with open(os.path.join(FILE_SAVE_PATH, id+'-mosaic'), 'wb') as f:
            f.write(obfs_image_bytes)
    except Exception as e:
        logger.error(f'convert_and_save: rollback: {e}')
        db.delete_image(id)
        try:
            os.remove(os.path.join(FILE_SAVE_PATH, id))
        except:
            pass
        try:
            os.remove(os.path.join(FILE_SAVE_PATH+'-mosaic', id))
        except:
            pass
```

The child Thread would execute `convert_and_save()` function. The logic is simple, but what we have to check is that if we provide image_url the server request it with timeout=3.

And if the uploaded file is not an image the except logic deletes image and the correspond row.

# Vulnerability

### Race Condition

```python
def convert_and_save(id, file=None, url=None):
    try:
        if url:
            res = requests.get(url, timeout=3)
```

The race condition arises at this code.

We have the child thread send web requests while the main thread sleeps for 5 seconds and disconnect if it takes longer than 3 seconds, but this can be bypassed.

If we make the server to infinite redirect(A⇒B, B⇒A) the server waits for more that 3 seconds.

### Scenario

So we can make race condition, and here is attack scenario

1. Request a flag image
    
    ⇒ the flag image would be saved with random id and the id would be inserted into the database.
    
2. access to the mosaicked flag image
    
    ⇒ after 10 seconds, the id of flag image would be deleted.
    
    ⇒ So we can upload and set the password with the flag id.
    
3. Upload image_url with same id of flag image
    
    ⇒ This results in infinite redirection.
    
    ```python
    def __add_image(password, id_, file=None, image_url=None, admin=False):
        t = Thread(target=convert_and_save, args=(id_, file, image_url))
        t.start()
    
        # no need, but time to waiting heavy response makes me excited!!
        if not admin:
            time.sleep(5)
    
    		....
    
        **db.add_image(id_, mimetype, password)**
    
        return urljoin(URLBASE, id_)
    ```
    
    - So the `db.add_image()` would be executed `even if the image wasn’t sended to the server!`
4. Access to the mosaicked image and pass the password and get the flag!