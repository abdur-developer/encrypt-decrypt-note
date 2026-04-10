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

    if ($combined === false) {
        throw new Exception("Invalid Base64");
    }

    if (strlen($combined) < 28) {
        throw new Exception("Data too short");
    }

    $iv = substr($combined, 0, 12);
    $ciphertextWithTag = substr($combined, 12);

    $tag = substr($ciphertextWithTag, -16);
    $ciphertext = substr($ciphertextWithTag, 0, -16);

    if (strlen($iv) !== 12 || strlen($tag) !== 16) {
        throw new Exception("Invalid IV or TAG");
    }

    $decrypted = openssl_decrypt(
        $ciphertext,
        'aes-256-gcm',
        $keyBytes,
        OPENSSL_RAW_DATA,
        $iv,
        $tag
    );

    if ($decrypted === false) {
        throw new Exception("Decryption failed (key/iv/tag mismatch)");
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

## ✅ JavaScript Version (AES-GCM + SHA-256)

```javascript
async function encryptData(plainText, apiPassword) {
    // 1. Text → Uint8Array
    const encoder = new TextEncoder();
    const data = encoder.encode(plainText);

    // 2. SHA-256 দিয়ে key generate
    const passwordBytes = encoder.encode(apiPassword);
    const keyHash = await crypto.subtle.digest("SHA-256", passwordBytes);

    // 3. AES key তৈরি
    const key = await crypto.subtle.importKey(
        "raw",
        keyHash,
        { name: "AES-GCM" },
        false,
        ["encrypt"]
    );

    // 4. Random IV (12 bytes)
    const iv = crypto.getRandomValues(new Uint8Array(12));

    // 5. Encrypt
    const encrypted = await crypto.subtle.encrypt(
        {
            name: "AES-GCM",
            iv: iv,
            tagLength: 128
        },
        key,
        data
    );

    // 6. IV + encrypted combine
    const combined = new Uint8Array(iv.length + encrypted.byteLength);
    combined.set(iv, 0);
    combined.set(new Uint8Array(encrypted), iv.length);

    // 7. Base64 encode
    return btoa(String.fromCharCode(...combined));
}
```

---

## 📌 ব্যবহার উদাহরণ

```javascript
encryptData("Hello World", "your_api_password")
    .then(result => console.log(result));
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
