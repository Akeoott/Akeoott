<style>
    body {
        font-family: sans-serif;
        font-size: 1.25em;
        background-color: #0d1117;
    }
</style>

---

# **GPG-Encrypted Archive Guide**  
### *Compress → Encrypt → Decrypt (Using Existing GPG Key)*  

```bash
# Install GPG (if missing)
sudo apt install gnupg tar zstd xz-utils  # Debian/Ubuntu
sudo pacman -S gnupg tar zstd xz         # Arch
sudo dnf install gnupg tar zstd xz       # Fedora
```

---

## **1. Compress the File**  
### **Option A: Fast Compression (`.tar.zst`)**  
```bash
tar cvf - my_important_dir | zstd -9 -o backup.tar.zst
```
- **`my_important_file.txt`** → Your input file.  
- **`backup.tar.zst`** → Output (compressed).  

### **Option B: Maximum Compression (`.tar.xz`)**  
```bash
tar cvf - my_important_dir | xz -9 -T0 -c > backup.tar.xz
```
- **`-T0`**: Use all CPU threads.  

---

## **2. Encrypt with GPG (Using Existing Key)**  
### **Step 1: Find Your Key**  
```bash
gpg --list-keys
```
Example output:  
```
pub   rsa4096 2023-01-01 [SC]
      1234ABCD...  # <-- THIS IS YOUR KEY ID
uid       [ultimate] Your Name <your@email.com>
```

### **Step 2: Encrypt**  
```bash
gpg --encrypt --recipient "Your Name" --output backup.tar.zst.gpg backup.tar.zst
```
- **`backup.tar.zst`** → Compressed file from Step 1.  
- **`backup.tar.zst.gpg`** → Encrypted output.  

---

## **3. Decrypt the File**  
### **Step 1: Ensure Key is Imported**  
*(Skip if already done)*  
```bash
gpg --import private_key.asc  # Replace with your key file
```

### **Step 2: Decrypt (One-Liner)**  
```bash
gpg --decrypt backup.tar.zst.gpg > decrypted_backup.tar.zst

# Alternative:
# Use if above fails to decrypt or if your not on a desktop enviorment.
gpg --batch --passphrase "your_password" --decrypt backup.tar.zst.gpg > decrypted_backup.tar.zst
```
- **`backup.tar.zst.gpg`** → Encrypted file.  
- **`decrypted_backup.tar.zst`** → Output (compressed, ready for extraction).  

---

### **Key Notes**  
🔐 **Security**:  
- Never hardcode passwords in scripts (use `--passphrase-file` instead).  
- Delete temporary files after decryption:  
  ```bash
  shred -u backup.tar.zst
  ```

⚡ **Need Automation?**  
```bash
# Decrypt + Decompress in One Step (No Intermediate File)
gpg --batch --passphrase "your_password" --decrypt backup.tar.zst.gpg | zstd -d -c > my_important_dir
```

---

### **Troubleshooting**  
❌ **"gpg: decryption failed"** → Wrong password/key.  
❌ **"No such file"** → Check paths/filenames.