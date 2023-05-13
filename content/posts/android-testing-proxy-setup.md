+++
author = "Anthony Cozamanis"
title = "Android Testing Proxy Setup"
date = "2023-05-13"
description = "A very brief write up on setting up an AVD to make us of a web proxy for testing."
tags = [
    "owsap",
    "zap",
    "burp",
    "android",
    "pentesting",
]
series = ["OWASP ZAP"]
+++
*Note:* Setup the Android device like normal in Android studio. Get the device ID:
```bash
cd ${ANDROID_SDK}/tools/emulator
./emulator -list-avds
```
The ID will look like `Nexus_6_API_26`

## Setup TLS
Export the CA from either Burp or ZAP (which ever you will be using) and rename to an Android friendly format, the cert hash plus string ".0"

### Burp
```bash
cp ca.der $(openssl x509 -inform DER -subject_hash_old -in  ca.der | head -n 1)".0"
```

### ZAP
```bash
cp owasp_zap_root_ca.cer $(openssl x509 -inform PEM -subject_hash_old -in  owasp_zap_root_ca.cer | head -n 1)".0"
```

### Start emulator and push the certificate
Example will be for AVD of name Nexus_6_API_26 and certificate named 608ecb18.0
```bash
./emulator -netdelay none -netspeed full -no-snapstorage -avd Nexus_6_API_26 -writable-system
adb root
adb remount
adb push 608ecb18.0 /system/etc/security/cacerts/
```

## Install the Application
Get the device ID and make note of it, looks like `emulator-####`
```bash
adb devices
```
Then push the APK
```bash
adb -s emulator-#### install application.apk
```

## Quirks
* Some applications don't respect the AVD proxy settings, in some case you need to set the proxy location in the emulator.
