+++
author = "Anthony Cozamanis"
title = "User Enumeration and Bruteforcing With OWASP ZAP"
date = "2023-05-13"
description = "In this brief piece, you'll learn a way to perform user enumeration and bruteforcing with OWASP ZAP."
tags = [
    "owsap",
    "zap",
    "pentesting",
]
series = ["OWASP ZAP"]
+++

So, you've been told to (legally) break into a WordPress website. Where to start? You can't go wrong investing a bit of time trying to break into legitmate user accounts. Here's how you can do that with ZAP.

Target website: https://192.168.122.34

![Screenshot_20230513_183404.png](/img/Screenshot_20230513_183404.png)

There's a few way to enumerate users on a WordPess blog and `wpscan` does these easily. We'll be taking a slight different approach and target the password reset page (https://192.168.122.34/wp-login.php?action=lostpassword) and the login page (https://192.168.122.34/wp-login.php).

The password reset page for WordPress blogs is very user-friendly, for users *and* attackers. It provides visitors to the page helpful clues if a particular user exists. For example, have a look at the message received when we put an invalid username into the "Username or Email Address" field:

![Screenshot_20230513_184058.png](/img/Screenshot_20230513_184058.png)

The POST request returns a Status code 200 sigifying that the page has just reloaded as seen by the following ZAP screenshot:

![Screenshot_20230513_184617.png](/img/Screenshot_20230513_184617.png)

As you can see, it's told us that there's no reason to bruteforce passwords for that username as it doesn't exist. Now if we put a valid username, the result is dramatically different:

![Screenshot_20230513_184248.png](/img/Screenshot_20230513_184248.png)

Additionally, the POST request returns a Status code 302 sigifying that the page has redirected us (presumably) to another page to tell us the password reset process has commenced:

![Screenshot_20230513_185007.png](/img/Screenshot_20230513_185007.png)

Rather than manually punch in usernames one at a time, we can automate this process with ZAP's fuzzer. To demonstrate we use make use of this user list:

```
root
test
guest
info
adm
mysql
user
administrator
oracle
ftp
pi
puppet
ansible
admin
ec2-user
vagrant
azureuser
```

To setup up the attack, we want to highlight the incorrect username in the request, right-click and select "Fuzz...":

![Screenshot_20230513_185437.png](/img/Screenshot_20230513_185437.png)

Add in our list of usernames by clicking "Payloads...":

![Screenshot_20230513_195425.png](/img/Screenshot_20230513_195425.png)

Then click "Start Fuzzer":

![Screenshot_20230513_195447.png](/img/Screenshot_20230513_195447.png)

And there we have it, the Fuzz has revealed a correct username for us to target, `admin`:

![Screenshot_20230513_195701.png](/img/Screenshot_20230513_195701.png)

Now we can turn our attention to the login page. What we want to do is attempt a login so we have a request to work with for the next Fuzz:

![Screenshot_20230513_200551.png](/img/Screenshot_20230513_200551.png)

As we did before, we will highlight the incorrect password in the request, select Fuzz, select "Payloads...", add our password list and click "Start Fuzzer":

![Screenshot_20230513_200831.png](/img/Screenshot_20230513_200831.png)

Here's my password list for the sake of this example:
```
password
123456
123456789
qwerty
password
12345
12345678
111111
1234567
123123
qwerty123
1q2w3e
1234567890
DEFAULT
Password123!
000000
abc123
654321
123321
qwertyuiop
Iloveyou
666666
```

After we run the Fuzz, we (should) have a correct password. `Password123!`, very secure:

![Screenshot_20230513_200846.png](/img/Screenshot_20230513_200846.png)

And we are in!

![Screenshot_20230513_200928.png](/img/Screenshot_20230513_200928.png)

