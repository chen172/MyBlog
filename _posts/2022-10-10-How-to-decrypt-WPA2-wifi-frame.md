---

layout: post
title: "How to Decrypt WPA2 Wi-Fi Frames"

---

### I. Identify the Frame Type We Need

All 802.11 frames fall under one of three categories: **Management**, **Control**, or **Data** frames. For WPA2 decryption, we only need to focus on **Data frames**.

The specific type of Data frame we’re concerned with is called **QoS Data**, with a type/subtype value of `0x0028`. In Wireshark, these frames are displayed under the label "QoS Data":

![QoS](/MyBlog/assets/picture/QoS.png){\:class="img-responsive"}

We can see that there are two protocol types used in these frames:

1. **EAPOL** - This is used for the **Four-Way Handshake**.
2. **802.11** - This is used for **encrypted Wi-Fi data**.

In this process, we’ll focus on the **802.11** frames that contain the encrypted data we need to decrypt.

---

### II. Decrypting WPA2 Data

To decrypt WPA2-encrypted Wi-Fi data, we need to follow a series of steps. Here’s a breakdown of the necessary process:

---

#### 1. **Derive the PMK (Pairwise Master Key)** from the passphrase and ESSID (Wi-Fi network name)

The **PMK** is derived from the Wi-Fi passphrase (shared key) and the **ESSID** (Wi-Fi network name). This is typically done using PBKDF2 (Password-Based Key Derivation Function 2), and it forms the base key for subsequent calculations.

---

#### 2. **Extract the Nonces** from the Four-Way Handshake

The next step is to extract the **Supplicant Nonce (SNonce)** and the **Authenticator Nonce (ANonce)** from the four-way handshake frames. You only need two of the four handshake frames, which contain the nonces for both the client (supplicant) and the access point (authenticator).

---

#### 3. **Derive the PTK (Pairwise Transient Key)**

The **PTK** is derived using the following formula:

```
PTK = PRF(PMK || ANonce || SNonce || AMAC || SMAC)
```

Where:

* **PRF** is a Pseudorandom Function.
* **PMK** is the Pairwise Master Key.
* **ANonce** and **SNonce** are the nonces from the four-way handshake.
* **AMAC** and **SMAC** are the MAC addresses of the authenticator and supplicant, respectively.

To calculate the **PTK**, we use **HMAC-SHA1** to encrypt the concatenated values of PMK, ANonce, SNonce, AMAC, and SMAC:

```c
int i;
unsigned char pke[100];

// "Pairwise key expansion" is used as the label for the PRF
memcpy(pke, "Pairwise key expansion", 23);

// Append the MAC addresses and nonces to the key expansion structure
memcpy(pke + 23, wpa->stmac, 6);        // Supplicant MAC
memcpy(pke + 29, wpa->bssid, 6);        // BSSID
memcpy(pke + 35, wpa->snonce, 32);      // SNonce
memcpy(pke + 67, wpa->anonce, 32);      // ANonce

// Calculate PTK using HMAC-SHA1
for (i = 0; i < 4; i++) {
    pke[99] = (uint8_t) i;
    HMAC(EVP_sha1(), pmk, 32, pke, 100, wpa->ptk + i * 20, NULL);
}
```

---

#### 4. **Verify the PTK** by comparing it with the MIC (Message Integrity Code)

Once the PTK is derived, we need to verify it by comparing it to the **MIC**. The **MIC** is a checksum used to ensure data integrity, and it’s part of the four-way handshake.

To verify, we use HMAC-SHA1 to encrypt the **EAPOL data** using the **PMK**:

```c
HMAC(EVP_sha1(), wpa->ptk, 16, wpa->eapol, wpa->eapol_size, mic, NULL);
```

---

#### 5. **Decrypt the Data (CCMP using AES-CTR-MAC)**

Once the PTK has been verified, we can proceed to decrypt the **CCMP** (Counter Mode with Cipher Block Chaining Message Authentication Code Protocol) encrypted data. CCMP uses **AES in CTR mode** to encrypt the data packets.

Decryption typically involves using the derived PTK as the key in the AES decryption algorithm to decrypt the ciphertext.

---

### III. Conclusion

Decrypting WPA2 frames involves several important steps, including deriving the PMK, extracting nonces from the four-way handshake, calculating the PTK, verifying the integrity of the PTK with the MIC, and finally decrypting the data using AES. By following these steps, you can successfully decrypt encrypted WPA2 data frames.

---

### References

1. [Aircrack-ng - airdecap-ng.c](https://github.com/aircrack-ng/aircrack-ng/blob/master/src/airdecap-ng/airdecap-ng.c)
2. [Aircrack-ng - crypto.c](https://github.com/aircrack-ng/aircrack-ng/blob/master/lib/crypto/crypto.c)
3. [OpenSSL HMAC Documentation](https://www.openssl.org/docs/man1.1.1/man3/HMAC.html)
4. [SHA1-based HMAC Implementation](https://github.com/chen172/crypto/blob/main/hmac.c)
