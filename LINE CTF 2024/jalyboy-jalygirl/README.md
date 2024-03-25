# jalyboy-jalygirl

| framework | Spring + Java |
| --- | --- |
| vulnerability | CVE-2022-21449 |

# Description

The source code closely resembles that of the "jalyboy-baby” challenge.

The key difference is that this challenge employs the ES Algorithm for generating and verifying JWT signatures.

# Exploit

![Untitled](jalyboy-jalygirl%2053b51473c80f4817b53c8dd25646fc3b/Untitled.png)

The exception handling was the same as in the previous challenge, so I decided to utilize tools provided by a Burp extension.

Fortunately, I obtained the flag without prior knowledge of **`CVE-2022-21449`**.

This vulnerability is well-known, hence the Burp extension includes a tool specifically for testing it.