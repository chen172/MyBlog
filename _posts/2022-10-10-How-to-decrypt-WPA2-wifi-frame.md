---
layout: post
title: "How to decrypt WPA2 wifi frame"
---

### I. Find the frame format we need
All 802.11 frames fall under one of the three types: management, control, or data. But we only need ```data frame```, the other two types actually just exchange device
information. The ```data frame``` we need calls ```QoS data```, type/subtype is 0x0028. Wireshark shows ```QoS data``` in the following:

![QoS](/MyBlog/assets/picture/QoS.png){:class="img-responsive"}

We can see there are two types of protocol, ```EAPOL and 802.11```. Acutally, ```EAPOL``` is for fourway handshakes, ```802.11``` is for encrypted wifi data.

### II. decrypt WPA2 data
#### 1. derive the PMK from the passphrase and the essid(wifi name)
#### 2. derive supplicant nonce and authenticator nonce from four-way handshake frame(actually you just need two of the four)
#### 3. derive the PTK(pairwise transcient keys) from a bunch of stuff
PTK is calculated by ```PTK = PRF(PMK || ANonce || SNonce || AMAC || SMAC)```, we use ```sha1 based HMAC``` encrypt ```(PMK || ANonce || SNonce || AMAC || SMAC)``` by ```PMK```.
```c
  int i;
  unsigned char pke[100];
  
  memcpy(pke, "Pairwise key expansion", 23);

  memcpy(pke + 23, wpa->stmac, 6);
  memcpy(pke + 29, wpa->bssid, 6);

  memcpy(pke + 35, wpa->snonce, 32);
  memcpy(pke + 67, wpa->anonce, 32);

  // calculate the PTK by PTK = PRF(PMK || ANonce || SNonce || AMAC || SMAC)
  for (i = 0; i < 4; i++) {
     pke[99] = (uint8_t) i;
     HMAC(EVP_sha1(), pmk, 32, pke, 100, wpa->ptk + i * 20, NULL);
  }
```
Then check if we get the right ```PTK``` by compare with ```MIC(Message Integrity Code)```, correct ```MIC``` is from fourway handshake.
Again we use ```sha1 based HMAC``` encrypt ```EAPOL data``` by ```PMK```.
```c
HMAC(EVP_sha1(), wpa->ptk, 16, wpa->eapol, wpa->eapol_size, mic, NULL);
```
#### 4. decrypt CCMP(AES-CTR-MAC)

##### reference
1. https://github.com/aircrack-ng/aircrack-ng [airdecap-ng.c](https://github.com/aircrack-ng/aircrack-ng/blob/master/src/airdecap-ng/airdecap-ng.c) and [crypto.c](https://github.com/aircrack-ng/aircrack-ng/blob/master/lib/crypto/crypto.c)
2. https://www.openssl.org/docs/man1.1.1/man3/HMAC.html
3. [sha1 based HMAC implementation](https://github.com/chen172/crypto/blob/main/hmac.c)


