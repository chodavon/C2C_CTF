
# This is the C2C CTF Write Up 
## Cho Davon
## Blockchain Convergence

1. Solve the challenge in the browser and submit your solutions.
2. Click "Launch" to start the instance.
3. Copy the provided `RPC_URL`, `Private Key`, and `Setup Address`.
4. Run the provided script:
	```bash
	python3 solve_convergence.py
	```
5. When prompted, input the `RPC_URL`, `Private Key`, and `Setup Address`.
6. After successful execution, retrieve the flag from the challenge portal:  
	[http://challenges.1pc.tf:51788/ad7cc1e1-9d4d-4a95-b9c3-4dbd926ace02](http://challenges.1pc.tf:51788/ad7cc1e1-9d4d-4a95-b9c3-4dbd926ace02)

---

## Blockchain TGE

1. Solve the challenge in the browser and submit your solutions.
2. Click "Launch" to start the instance.
3. Copy the `RPC_URL`, `Private Key`, and `Setup Address`.
4. Run the provided script:
	```bash
	python3 solve_tge.py
	```
5. Enter the `RPC_URL`, `Private Key`, and `Setup Address` when prompted.
6. Upon completion, obtain the flag from the challenge portal.

---

## Misc Welcome

1. Click the link provided in the spam email.
2. Enter the given username and password.
3. The flag will be displayed upon successful login.

---

## Misc JinJail

1. Create a new instance for the challenge.
2. Download the provided ZIP file.
3. Run the script (ensure the service is running on port 32157).
4. Access the service to retrieve the flag.

---

## Forensic Log

1. Create a new instance and copy the connection details.
2. Connect using netcat:
	```bash
	nc <instance_address> <port>
	```
3. Answer the questions (1-9) using tools such as `tcpdump` and `nmap` as needed.
4. Upon completion, the flag will be displayed.

---

## Reverse Bunaken

### Introduction

I started by examining the two files provided: the Linux executable (`bunaken`) and the encrypted text file (`flag.txt.bunakencrypted`). It was immediately apparent that the executable was a custom ransomware or encryption tool used to lock the flag, so I needed to reverse its logic to recover the original flag.

### Finding the Hardcoded Password

Before using advanced tools like Ghidra, I performed basic static analysis with `strings` on the binary. Among the output, the word `sulawesi` stood out as suspicious and likely to be the hardcoded password used for encryption. This was confirmed by a clue labeled as clue 1.

### Extracting the IV

Next, I investigated how the Initialization Vector (IV) was handled. In many CTFs, encryption tools prepend a randomly generated 14- to 16-byte IV to the start of the encrypted file. I assumed the first 16 bytes of the base64-decoded file were the IV, with the remainder being the ciphertext.

### Key Derivation

Assuming the cipher was AES-128-CBC, I noted that AES-128 requires a 16-byte key, but `sulawesi` is only 8 bytes. The binary likely hashes the password to stretch it. Since SHA-256 outputs 32 bytes, I theorized the program takes the SHA-256 hash of `sulawesi` and uses the first 16 bytes as the AES-128 key.

### Decryption Script

With all the pieces in place (cipher, IV, and key derivation), I wrote the following Node.js script to decrypt the file:

```js
const crypto = require("crypto");

const key = "sulawesi";
const data = Buffer.from("3o2Gh52pjRk80IPViTp8KUly+kDGXo7qAlPo2Ff1+IOWW1ziNAoboyBZPX6R4JvNXZ4iWwc662Nv/rMPLdwrIb3D4tTbOg/vi0NKaPfToj0=", "base64");

const iv = data.subarray(0, 16);
const enc = data.subarray(16);

const hash = crypto.createHash("sha256").update(key).digest().subarray(0, 16);

const decipher = crypto.createDecipheriv("aes-128-cbc", hash, iv);
let out = decipher.update(enc);
out = Buffer.concat([out, decipher.final()]);

console.log(out.toString());
```

**Note:** Run this script with `node script.js` in the directory containing the files.

### Result

Running the script successfully decrypted the file and revealed the flag.

---
