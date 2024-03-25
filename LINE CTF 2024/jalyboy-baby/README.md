# jalyboy-baby

| framework | Spring + Java |
| --- | --- |
| vulnerability | JWT none signing attack |

# Description

In order to obtain the flag, we must bypass JWT authentication.

The server uses the **`jsonwebtoken`** library, but it was not efficient to dig into the library's source code.

So, our goal is to impersonate an ADMIN.

# Exploit

```kotlin
try {
    Jwt jwt = Jwts.parser().setSigningKey(secretKey).parse(j);
    Claims claims = (Claims) jwt.getBody();
    if (claims.getSubject().equals(ADMIN)) {
        sub = ADMIN;
    } else if (claims.getSubject().equals(GUEST)) {
        sub = GUEST;
    }
} catch (Exception e) {
//            e.printStackTrace();
}
```

In the provided source code, the `exception handling` for JWT parsing logic may be incorrect.

![Untitled](jalyboy-baby%20db4102948add4d398b6a50ee08d8d330/Untitled.png)

Given this potential vulnerability, I decided to employ well-known JWT attacks, specifically the none signing attack.

And that's how I obtained the flag!