# 🔐 AES-256-GCM Encryption (Java ↔ PHP Compatible)

---

# ☕ Java Code

## ✅ Encrypt + Decrypt

```java
    import javax.crypto.Cipher;
    import javax.crypto.SecretKey;
    import javax.crypto.spec.GCMParameterSpec;
    import javax.crypto.spec.SecretKeySpec;
    import java.nio.charset.StandardCharsets;
    import java.security.MessageDigest;
    import java.security.SecureRandom;
    import java.util.Base64;
    
    public class AesGcmUtil {
    
        private static final String SECRET_KEY_STRING = "AbdurRahman123$$";
        private static final int GCM_IV_LENGTH = 12;          // bytes
        private static final int GCM_TAG_LENGTH = 128;        // bits (16 bytes)
    
        private static SecretKey getKey() throws Exception {
            MessageDigest sha = MessageDigest.getInstance("SHA-256");
            byte[] keyBytes = sha.digest(SECRET_KEY_STRING.getBytes(StandardCharsets.UTF_8));
            return new SecretKeySpec(keyBytes, "AES");
        }
    
        public static String encrypt(String plainText) throws Exception {
            if (plainText == null) return null;
    
            SecretKey secretKey = getKey();
            Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
    
            byte[] iv = new byte[GCM_IV_LENGTH];
            new SecureRandom().nextBytes(iv);
    
            GCMParameterSpec spec = new GCMParameterSpec(GCM_TAG_LENGTH, iv);
            cipher.init(Cipher.ENCRYPT_MODE, secretKey, spec);
    
            byte[] encrypted = cipher.doFinal(plainText.getBytes(StandardCharsets.UTF_8));
    
            // Combine IV + ciphertext+tag
            byte[] combined = new byte[iv.length + encrypted.length];
            System.arraycopy(iv, 0, combined, 0, iv.length);
            System.arraycopy(encrypted, 0, combined, iv.length, encrypted.length);
    
            return Base64.getEncoder().encodeToString(combined);
        }
    
        public static String decrypt(String encryptedBase64) throws Exception {
            if (encryptedBase64 == null) return null;
    
            SecretKey secretKey = getKey();
            byte[] combined = Base64.getDecoder().decode(encryptedBase64);
    
            if (combined.length < GCM_IV_LENGTH + 16) {   // at least IV + tag (empty ciphertext)
                throw new IllegalArgumentException("Invalid ciphertext length");
            }
    
            byte[] iv = new byte[GCM_IV_LENGTH];
            System.arraycopy(combined, 0, iv, 0, GCM_IV_LENGTH);
    
            byte[] cipherText = new byte[combined.length - GCM_IV_LENGTH];
            System.arraycopy(combined, GCM_IV_LENGTH, cipherText, 0, cipherText.length);
    
            Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
            GCMParameterSpec spec = new GCMParameterSpec(GCM_TAG_LENGTH, iv);
            cipher.init(Cipher.DECRYPT_MODE, secretKey, spec);
    
            byte[] decrypted = cipher.doFinal(cipherText);
            return new String(decrypted, StandardCharsets.UTF_8);
        }
    }
```

---

# 🐘 PHP Code

## ✅ Encrypt + Decrypt

```php
    class AesGcm {
        private const SECRET_KEY = "AbdurRahman123$$";
        private const GCM_IV_LENGTH = 12;
        private const GCM_TAG_LENGTH = 16;
    
        private static function getKeyBytes(): string {
            return hash('sha256', self::SECRET_KEY, true);
        }
    
        public static function encrypt(string $plainText): string {
            $keyBytes = self::getKeyBytes();
            $iv = random_bytes(self::GCM_IV_LENGTH);
            $tag = '';
    
            $cipherText = openssl_encrypt(
                $plainText,
                'aes-256-gcm',
                $keyBytes,
                OPENSSL_RAW_DATA,
                $iv,
                $tag,
                '',
                self::GCM_TAG_LENGTH
            );
    
            if ($cipherText === false) {
                throw new RuntimeException('Encryption failed');
            }
    
            return base64_encode($iv . $cipherText . $tag);
        }
    
        public static function decrypt(string $encryptedData): string {
            $keyBytes = self::getKeyBytes();
            $combined = base64_decode($encryptedData, true);
    
            if ($combined === false || strlen($combined) < self::GCM_IV_LENGTH + self::GCM_TAG_LENGTH) {
                throw new InvalidArgumentException('Invalid ciphertext format');
            }
    
            $iv = substr($combined, 0, self::GCM_IV_LENGTH);
            $cipherTextWithTag = substr($combined, self::GCM_IV_LENGTH);
    
            $tag = substr($cipherTextWithTag, -self::GCM_TAG_LENGTH);
            $cipherText = substr($cipherTextWithTag, 0, -self::GCM_TAG_LENGTH);
    
            $decrypted = openssl_decrypt(
                $cipherText,
                'aes-256-gcm',
                $keyBytes,
                OPENSSL_RAW_DATA,
                $iv,
                $tag
            );
    
            if ($decrypted === false) {
                throw new RuntimeException('Decryption failed – key, IV or tag mismatch');
            }
    
            return $decrypted;
        }
    }
```

---

### Java তে কল

```java
public class Main {
    public static void main(String[] args) {
        try {
            // এনক্রিপ্ট করা
            String originalText = "1709409266";
            String encrypted = AesGcmUtil.encrypt(originalText);
            System.out.println("Encrypted: " + encrypted);
            
            // ডিক্রিপ্ট করা
            String decrypted = AesGcmUtil.decrypt(encrypted);
            System.out.println("Decrypted: " + decrypted);
            
            // চেক করা
            System.out.println("Match: " + originalText.equals(decrypted));
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### PHP

```php
<?php
require_once 'AesGcm.php';  // যদি আলাদা ফাইল হয়

// এনক্রিপ্ট করা
$originalText = "1709409266";
$encrypted = AesGcm::encrypt($originalText);
echo "Encrypted: " . $encrypted . "\n";

// ডিক্রিপ্ট করা
$decrypted = AesGcm::decrypt($encrypted);
echo "Decrypted: " . $decrypted . "\n";

// চেক করা
echo "Match: " . ($originalText === $decrypted ? "true" : "false") . "\n";
?>
```

---
# ✅ Class Reference
<p align="center">
  <img src="encrypt in java.png"  />
  <img src="encrypt with key in java.png"  />
  <img src="decrypt with key in java.png"  />
  <img src="encrypt decrypt php with key.png"  />
</p>
---
