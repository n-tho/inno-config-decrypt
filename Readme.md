# Innovaphone Configuration Decryptor

This is a 100% client-side HTML+JavaScript tool to analyze and decrypt Innovaphone **standard** configuration files.

## 🚀 Features

- Parses Innovaphone config files locally in the browser
- Decrypts system VARS and object passwords
- Live search to filter any result
- Export result to CSV or JSON

## 🛡 Security

- No data leaves your browser
- All processing is done locally
- Safe to use offline

## 📥 Usage

1. Open `index.html` in a browser (Chrome/Firefox/Edge)
2. Click **“Upload Configuration File”** and select a **standard** configuration file
3. The tool:
   - Detects the device type
   - Extracts and decrypts:
     - `PBX0/PWD`
     - PBX master key
     - `vars` entries
     - User passwords (`pwd=...`)
4. Filter results using the **search bar**
5. Export results using **CSV** or **JSON** buttons
# 🔐 Innovaphone Password Decryption Process

This document describes the technical steps used to decrypt passwords in Innovaphone PBX configuration files.

The configuration format stores various secrets (e.g., PBX password, user passwords) encrypted using RC4. Decryption requires a series of dependent steps with specific keys derived from known values.

---

## 🧩 Step-by-Step Decryption Flow

### 1. **Extract the Encrypted PBX Password**

In the config file, locate the following line:

```
vars create PBX0/PWD pc <encrypted_hex>
```

- This is the encrypted PBX system password (`PBX0/PWD`).
- It is RC4-encrypted with a key based on `"admin"` and the device type.

---

### 2. **Build the Initial Key for PBX0/PWD**

Create a 32-byte key like this:

```
key = pad("admin", 16) + pad(device_type, 16)
```

- `pad(value, 16)` fills the string with Null to reach 16 bytes.
- `device_type` is typically extracted from the config header (e.g., `"ip411"`).

Use this key to RC4-decrypt `PBX0/PWD`.

---

### 3. **Extract the PBX Master Key**

In the config the user `__ADMIN__` has the key attribute:

```
key="<hex_value>"
```

- This is the encrypted **PBX master key**.
- It is RC4-encrypted using the decrypted value of `PBX0/PWD`.

---

### 4. **Build the Key for PBX Master Key**

Create a 16-byte key:

```
key = pad(pbx_pwd, 16)
```

- Use the decrypted PBX0/PWD from Step 2.
- Decrypt the master key with this key using RC4.

---

### 5. **Decrypt User Passwords**

User entries are defined like this:

```
(cn=someuser)...(pbx=<user pwd="<hex_encrypted_pw>" ...>)
```

- The `pwd="..."` field contains the encrypted password.
- Only entries where `pwd` is in a `<user ...>` pbx block are relevant.

---

### 6. **Build the Key for User Passwords**

Create another 16-byte key:

```
key = pad(pbx_master_key, 16)
```

Use this to decrypt each user password (`pwd="..."`) using RC4.

---

## 📌 Summary of Decryption Keys

| Target           | Key Source                                | Key Format                      |
|------------------|--------------------------------------------|----------------------------------|
| PBX0/PWD         | `"admin"` + `device_type`                  | 32 bytes (2× padded 16-byte)     |
| PBX master key   | decrypted PBX0/PWD                         | 16 bytes padded                  |
| User passwords   | decrypted PBX master key                   | 16 bytes padded                  |

---

## 🔐 Notes

- If `pwd="000000...0000"` (all zeroes), it's a placeholder and should be treated as `(empty / dummy)`
- Some entries may have no password at all – skip those
- Only passwords inside `pbx=<user` tags are considered valid

---

## 🧪 Based On

Documentation:  
[Innovaphone Wiki: Howto Encrypt or Decrypt PBX user passwords](https://wiki.innovaphone.com/index.php?title=Howto:Encrypt_or_Decrypt_PBX_user_passwords)

## 📄 License

MIT License – Free for personal or professional use.
