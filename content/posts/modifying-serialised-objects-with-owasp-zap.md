+++
author = "Anthony Cozamanis"
title = "Modifying Serialised Objects With OWASP ZAP"
date = "2023-05-14"
description = "This post is about Modifying Serialised Objects With OWASP ZAP...surprise!"
tags = [
    "owsap",
    "zap",
    "pentesting",
]
series = ["OWASP ZAP"]
+++
*Note:* This was part of an overall effort to complete PortSwigger's Web Security Academy without Burp Suite.

Suppose you're conducting a security audit on an application that employs session cookies encoded with URL and Base64. Here's what you can do:

1. Log in to the application using your credentials. Observe that the GET request for the `/` endpoint includes a session cookie encoded with URL and Base64. Copy this request and open it in the OWASP ZAP Manual Request Editor. Also, highlight the value of the session cookie and right-click to open the context menu, then choose "Encode/Decode/Hash...".

2. URL-decode the value of the session cookie.

![a9b415d61d1653ca9d4d6b4e66efd5b0.png](/img/a9b415d61d1653ca9d4d6b4e66efd5b0.png)

3. Base64-decode the result obtained from step 2.

![525f4246577e70d96b91094cab85f71e.png](/img/525f4246577e70d96b91094cab85f71e.png)

4. Note that the `admin` attribute has a value of `b:0`, which represents the boolean value `false`. Change this to `b:1`.

5. Encode the updated session cookie value as follows:

![2be921ed5f2b3c30cc4052f727679d61.png](/img/2be921ed5f2b3c30cc4052f727679d61.png)

6. Then, URL-encode the result from step 5.

![6390308c8bf25bbca2f2c2c6b9e436a7.png](/img/6390308c8bf25bbca2f2c2c6b9e436a7.png)

7. In the request editor, replace this:

`Set-Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czo1OiJhZG1pbiI7YjowO30%3d`

with this:

`Set-Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czo1OiJhZG1pbiI7Yjox%0AO30%3D`

8. Send the modified request to the server.

![d6c7b71414f900c045f76a067379203a.png](/img/d6c7b71414f900c045f76a067379203a.png)

9. You'll see the link required to delete Carlos (`/admin/delete?username=carlos`). Use this link to delete his account.

10. Finally, resend the modified request to the server to carry out the deletion.

![fd0e7cb4a14f0f73f1246b7b589c89ef.png](/img/fd0e7cb4a14f0f73f1246b7b589c89ef.png)

