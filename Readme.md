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

## 🔐 Decryption Logic

The tool implements Innovaphone's password/key scheme:

1. `PBX0/PWD` is RC4-decrypted using:
key = pad("admin", 16) + pad(deviceType, 16)

2. The PBX master key (`key="..."`) is decrypted with:
key = pad(PBX0/PWD, 16)

3. User passwords are decrypted using:
key = pad(PBX master key, 16)


All operations follow Innovaphone's own method documented at:  
https://wiki.innovaphone.com/index.php?title=Howto:Encrypt_or_Decrypt_PBX_user_passwords


## 📄 License

MIT License – Free for personal or professional use.