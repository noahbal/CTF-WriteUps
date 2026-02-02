# Ocarina - Level 2
## Introduction
After proving her worth with the basic trials, Jeanne discovered that the most sacred melodies were protected by ancient mystical barriers. The divine songs now reach her ears in a transformed state. Their true beauty hidden behind layers of mystical protection that only the worthy can unravel.
## Challenge Resolution
We have a code similar to the first Ocarina challenge but now, the sequence comes encrypted, using this `crypto.js` file:
```js
// AES encryption implementation for client side
const AES_KEY = 'MyStr0ngSecretK3yForOcarin42024!';

const crypto = {
    // Convert string to bytes array
    stringToBytes(str) {
        return new TextEncoder().encode(str);
    },

    // Convert bytes array to string
    bytesToString(bytes) {
        return new TextDecoder().decode(bytes);
    },

    // Convert string to base64
    toBase64(str) {
        return btoa(str);
    },

    // Convert base64 to string
    fromBase64(str) {
        return atob(str);
    },

    // Import key for WebCrypto API
    async getKey() {
        return await window.crypto.subtle.importKey(
            'raw',
            this.stringToBytes(AES_KEY),
            { name: 'AES-CBC' },
            false,
            ['encrypt', 'decrypt']
        );
    },

    // Encrypt data
    async encrypt(data) {
        const key = await this.getKey();
        const iv = window.crypto.getRandomValues(new Uint8Array(16));
        const encrypted = await window.crypto.subtle.encrypt(
            { name: 'AES-CBC', iv },
            key,
            this.stringToBytes(data)
        );
        
        // Combine IV and encrypted data
        const combined = new Uint8Array(iv.length + encrypted.byteLength);
        combined.set(iv);
        combined.set(new Uint8Array(encrypted), iv.length);
        
        return this.toBase64(String.fromCharCode(...combined));
    },

    // Decrypt data
    async decrypt(encryptedData) {
        try {
            const key = await this.getKey();
            const data = new Uint8Array(
                [...this.fromBase64(encryptedData)].map(c => c.charCodeAt(0))
            );
            
            // Split IV and encrypted data
            const iv = data.slice(0, 16);
            const ciphertext = data.slice(16);
            
            const decrypted = await window.crypto.subtle.decrypt(
                { name: 'AES-CBC', iv },
                key,
                ciphertext
            );
            
            return this.bytesToString(new Uint8Array(decrypted));
        } catch (e) {
            console.error('Decryption error:', e);
            return '';
        }
    }
};
```
The encryption method here is `AES-CBC` with a block size of **16** and the main point of encrypting the incoming sequence is that we cannot send it back as is.
Indeed, the sequence is encrypted using a **random** `IV`, so the server will detect and close the websocket if we send back a sequence encrypted with the `same IV`. We are forced to decrypt and encrypt once again the sequence received from the server. To do this, I adapted the Python script from the first Ocarina challenge:
```python
#!/usr/bin/env python

from websockets.sync.client import connect
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
import base64, os, json

AES_KEY = b'MyStr0ngSecretK3yForOcarin42024!'
BLOCK_SIZE = 16

def encrypt_sequence(seq: str) -> str:
    iv = os.urandom(16)
    cipher = AES.new(AES_KEY, AES.MODE_CBC, iv)
    encrypted = cipher.encrypt(pad(seq.encode(), BLOCK_SIZE))
    combined = iv + encrypted
    return base64.b64encode(combined).decode()

def decrypt_sequence(encrypted_b64: str) -> str:
    # Base64 -> bytes
    combined = base64.b64decode(encrypted_b64)

    # Separate IV and ciphertext
    iv = combined[:16]
    ciphertext = combined[16:]

    # Decypher AES-CBC
    cipher = AES.new(AES_KEY, AES.MODE_CBC, iv)
    decrypted = cipher.decrypt(ciphertext)

    # Unpad and decode
    return unpad(decrypted, BLOCK_SIZE).decode()

def runGame():
    uri = "ws://ocarina2.web02.jeanne-hack-ctf.org/ws"
    with connect(uri) as websocket:
        round=0
        while round<100:
            message = websocket.recv()
            data = json.loads(message)

            if data["type"] == "sequence":
                seq = decrypt_sequence(data["seq"])
                round_ = data["round"]
                print(f"[+] Round {round_}: {seq}")

                # send back the encrypted sequence
                websocket.send(json.dumps({
                    "type": "verify",
                    "sequence": encrypt_sequence(seq)
                }))
                round = round_

            elif data["type"] == "result":
                if data["data"]["correct"]:
                    print("[✓] Correct")
                else:
                    print("[✗] Wrong")
            
            else:
                print("[!] Unexpected message:", data)

if __name__ == "__main__":
    runGame()
```
And once again, the flag is in the last message:
```flag
[!] Unexpected message: {'type': 'flag', 'data': {'flag': 'JDHACK{cryp70_0car1n4_m4st3r_w1th_AES_s3cr3ts!}', 'message': 'Completed 99 rounds!'}}
```

