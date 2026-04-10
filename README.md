# 🔐 AES-256-GCM Encryption (Java ↔ PHP Compatible)

এই README-তে Java এবং PHP ব্যবহার করে **AES-256-GCM encryption/decryption** সম্পূর্ণভাবে দেখানো হয়েছে, যাতে দুই প্ল্যাটফর্ম একে অপরের সাথে compatible থাকে।

---

## 📌 Configuration

- **Algorithm:** AES-256-GCM  
- **IV Length:** 12 bytes  
- **Auth Tag Length:** 16 bytes  
- **Key Derivation:** SHA-256  

---

## 🔑 Secret Key

```

AbdurRahman123$$

````

---

# ☕ Java Code

## ✅ Encrypt

```java
public static String encrypt(String plainText) throws Exception {
    String key = "AbdurRahman123$$";

    byte[] input = plainText.getBytes(StandardCharsets.UTF_8);

    MessageDigest sha = MessageDigest.getInstance("SHA-256");
    byte[] keyBytes = sha.digest(key.getBytes(StandardCharsets.UTF_8));
    SecretKeySpec secretKey = new SecretKeySpec(keyBytes, "AES");

    Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");

    byte[] iv = new byte[12];
    SecureRandom random = new SecureRandom();
    random.nextBytes(iv);

    GCMParameterSpec spec = new GCMParameterSpec(128, iv);
    cipher.init(Cipher.ENCRYPT_MODE, secretKey, spec);

    byte[] encrypted = cipher.doFinal(input);

    byte[] combined = new byte[iv.length + encrypted.length];
    System.arraycopy(iv, 0, combined, 0, iv.length);
    System.arraycopy(encrypted, 0, combined, iv.length, encrypted.length);

    return Base64.getEncoder().encodeToString(combined);
}
````

---

## ✅ Decrypt

```java
public static String decrypt(String encryptedBase64) throws Exception {
    String key = "AbdurRahman123$$";

    byte[] combined = Base64.getDecoder().decode(encryptedBase64);

    byte[] iv = new byte[12];
    System.arraycopy(combined, 0, iv, 0, 12);

    byte[] cipherText = new byte[combined.length - 12];
    System.arraycopy(combined, 12, cipherText, 0, cipherText.length);

    MessageDigest sha = MessageDigest.getInstance("SHA-256");
    byte[] keyBytes = sha.digest(key.getBytes(StandardCharsets.UTF_8));
    SecretKeySpec secretKey = new SecretKeySpec(keyBytes, "AES");

    Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
    GCMParameterSpec spec = new GCMParameterSpec(128, iv);

    cipher.init(Cipher.DECRYPT_MODE, secretKey, spec);

    byte[] decrypted = cipher.doFinal(cipherText);

    return new String(decrypted, StandardCharsets.UTF_8);
}
```

---

# 🐘 PHP Code

## ✅ Encrypt

```php
function encryptData($string) {
    $key = "AbdurRahman123$$";

    $keyBytes = hash('sha256', $key, true);
    $cipher = 'aes-256-gcm';

    $iv = random_bytes(12);
    $tag = '';

    $encrypted = openssl_encrypt(
        $string,
        $cipher,
        $keyBytes,
        OPENSSL_RAW_DATA,
        $iv,
        $tag,
        '',
        16
    );

    $combined = $iv . $encrypted . $tag;

    return base64_encode($combined);
}
```

---

## ✅ Decrypt

```php
function decryptData($encryptedData) {
    $key = "AbdurRahman123$$";

    $keyBytes = hash('sha256', $key, true);

    $combined = base64_decode($encryptedData);

    $iv = substr($combined, 0, 12);
    $ciphertextWithTag = substr($combined, 12);

    $tagLength = 16;

    $tag = substr($ciphertextWithTag, -$tagLength);
    $ciphertext = substr($ciphertextWithTag, 0, -$tagLength);

    $decrypted = openssl_decrypt(
        $ciphertext,
        'aes-256-gcm',
        $keyBytes,
        OPENSSL_RAW_DATA,
        $iv,
        $tag
    );

    if ($decrypted === false) {
        throw new Exception("Decryption failed");
    }

    return $decrypted;
}
```

---

# 🔄 Data Format

Encryption এর পর data structure হবে:

```
Base64(
    IV (12 bytes)
    + Ciphertext
    + Auth Tag (16 bytes)
)
```

---

# ⚠️ গুরুত্বপূর্ণ বিষয়

** প্রতিবার encryption-এ নতুন IV ব্যবহার করতে হবে
** Secret key কখনো public করা যাবে না
** AES-GCM নিজেই authentication দেয় (extra hash লাগবে না)

---

# 🧪 Example

### Java

```java
String enc = encrypt("1709409266");
String dec = decrypt(enc);
```

### PHP

```php
$enc = encryptData("1709409266");
$dec = decryptData($enc);
```

---

# ✅ Compatibility

| Platform | Encrypt | Decrypt |
| -------- | ------- | ------- |
| Java     | ✅       | ✅       |
| PHP      | ✅       | ✅       |

---

---
# ✅ Class Reference
<p align="center">
  <img src="encrypt in java.png"  />
  <img src="encrypt with key in java.png"  />
  <img src="decrypt with key in java.png"  />
  <img src="encrypt decrypt php with key.png"  />
</p>
---
