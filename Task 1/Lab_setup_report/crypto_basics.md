Cryptography Basics

Cryptography is used to protect data by converting it into a secure form so that unauthorized people cannot read it. It helps keep information safe when sending data over the internet.

Symmetric vs Asymmetric Encryption

Symmetric Encryption:
Symmetric encryption uses the same key to encrypt and decrypt data. Both the sender and receiver must know this secret key. It is fast and efficient, but the key must be shared safely, otherwise others may access the data.

Asymmetric Encryption:
Asymmetric encryption uses two keys: a public key and a private key. The public key is used to encrypt data, and the private key is used to decrypt it. It is more secure for communication because the private key is not shared, but it is slower than symmetric encryption.

Hashing (MD5, SHA256)

Hashing converts data into a fixed-length value called a hash. It is a one-way process, meaning the original data cannot be obtained from the hash.

MD5:
MD5 (Message Digest 5) is an older hashing method. It is not considered secure today because different data can sometimes produce the same hash.

SHA256:
SHA256 (Secure Hash Algorithm 256) is a stronger and more secure hashing method. Even a small change in the data will produce a completely different hash.

Hashing is mainly used to check data integrity and verify that data has not been changed.

Digital Certificates & SSL/TLS

SSL (Secure Sockets Layer) and TLS (Transport Layer Security) are protocols used to secure communication between a web browser and a website by encrypting the data.

Digital certificates are used to verify the identity of websites. They connect a public key with the website’s identity and are issued by trusted organizations called Certificate Authorities (CA). When a website is secure, a padlock symbol appears in the browser address bar.

Hands-on: OpenSSL

OpenSSL is a command-line tool used for encryption and decryption.

To encrypt a file:

openssl enc -aes-256-cbc -in input_file.txt -out encrypted_file.enc

To decrypt a file:

openssl enc -d -aes-256-cbc -in encrypted_file.enc -out decrypted_file.txt
