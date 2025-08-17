<style>
    body {
        font-family: sans-serif;
        font-size: 1.25em;
        background-color: #0d1117;
    }
</style>

---

# **Secure GPG Encryption Guide**  
### *Compress → Encrypt → Decrypt (Using GPG Keys Properly)*  

```bash
# Install required tools
sudo apt install gnupg tar zstd xz-utils  # Debian/Ubuntu
sudo pacman -S gnupg tar zstd xz         # Arch
sudo dnf install gnupg tar zstd xz       # Fedora
```

---

## **1. Compress the Data**  
### **Fast Compression (`.tar.zst`)**  
```bash
tar cvf - my_important_dir | zstd -9 -o backup.tar.zst
```

### **Maximum Compression (`.tar.xz`)**  
```bash
tar cvf - my_important_dir | xz -9 -T0 -c > backup.tar.xz
```

---

## **2. Encrypt with GPG**  
### **Find Your Key ID**  
```bash
gpg --list-keys --keyid-format LONG

# If key is from github:
gpg --list-secret-keys --keyid-format LONG
```
Look for:  
```
pub   rsa4096/ABCD1234ABCD1234 2023-01-01 [SC]
      1234ABCD...  # This is your Key ID
uid                 [ultimate] Your Name <your@email.com>
```

### **Encrypt Using Key ID (Recommended)**  
```bash
gpg --encrypt --recipient "ABCD1234ABCD1234" --output backup.tar.zst.gpg backup.tar.zst
```

### **Encrypt Using Email (Alternative)**  
```bash
gpg --encrypt --recipient "your@email.com" --output backup.tar.zst.gpg backup.tar.zst
```

---

## **3. Decrypt the File**  
### **Basic Decryption**  
```bash
gpg --decrypt backup.tar.zst.gpg > decrypted_backup.tar.zst
```

### **If Using Passphrase-Protected Key**  
```bash
gpg --batch --pinentry-mode loopback --passphrase "your_passphrase" --decrypt backup.tar.zst.gpg > decrypted_backup.tar.zst
```

---

## **Security Notes**  
- 🔐 **Always use Key IDs** instead of emails for encryption  
- 🚫 **Never use `-c` (symmetric)** for important backups  
- 📛 **Avoid plaintext passphrases** in commands (use `--passphrase-file` if needed)  
- 🗑️ **Clean up temporary files**:  
  ```bash
  shred -u backup.tar.zst decrypted_backup.tar.zst
  ```

---

## **Troubleshooting**  
- 🔍 **"No secret key"**: Import your private key with `gpg --import key.asc`  
- ❌ **Decryption fails**: Verify you're using the correct Key ID and passphrase  
- ⚠️ **GUI prompts appear**: Add `--pinentry-mode loopback` to force CLI mode
