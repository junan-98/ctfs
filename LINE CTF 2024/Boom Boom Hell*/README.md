# Boom Boom Hell*

![Untitled](Boom%20Boom%20Hell%2025ea94a2ba1c4142bf8fc8557da47519/Untitled.png)

| framework | bun & nodejs |
| --- | --- |
| vulnerability | command injection |

# Description

```jsx
const url = new URL(req.url);
if (url.pathname === "/chall") {
    const params = qs.parse(url.search, {ignoreQueryPrefix: true});
    if (params.url.length < escapeHTML(params.url).length) {    // dislike suspicious chars
        return new Response("sorry, but the given URL is too complex for me");
    }

    const lyURL = new URL(params.url, "https://www.lycorp.co.jp");
    if (lyURL.origin !== "https://www.lycorp.co.jp") {
        return new Response("don't you know us?");
    }

    console.log("[+] cmd:", `curl -sL ${lyURL}`);
    const rawFetched = await $`curl -sL ${lyURL}`.text();
```

This challenge parse the given url query parameter and insert the parameter to the shell command.

# Exploit

The origin check was existed, and the given paramter is processed as URL, I used %3f to bypass the url encoding.

Some of the characters were encdoed(I think) but it can be bypassed via, ``

### payload

![Untitled](Boom%20Boom%20Hell%2025ea94a2ba1c4142bf8fc8557da47519/Untitled%201.png)

![Untitled](Boom%20Boom%20Hell%2025ea94a2ba1c4142bf8fc8557da47519/Untitled%202.png)

It was not possible to do the RCE since it was too complicated, but we can inject more options to the curl command.