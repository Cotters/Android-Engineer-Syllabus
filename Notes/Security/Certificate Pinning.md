Referred to as 'SSL pinning' in Starling docs. 
A better, more accurate and correct term is TLS Pinning or Certificate Pinning.

## SSL (deprecated) & TLS

These are just web protocols that establish secure, encrypted communication between nodes in a network. They use Public Key Encryption to encrypt messages to prevent snooping. **TLS built upon and superseded SSL which is now deprecated.**
### Analogy

Imagine Adam has a lock and key. He keeps the key 'private' in his pocket, but can and does make copies of his lock which he willingly hands out to anyone who wants one.
Then, if someone wants to send a message to Adam, they can put it in a container, and lock the container with Adam's lock. Adam, who is the only person with the key to this lock, can unlock the lock and open the container to read the message inside. This is SSL/TLS.

However, there is a flaw here where someone can intercept the traffic and alter, drop, or send new traffic mimicking the sender. This is called Man in the Middle (MITM) attacks.

## Man in the Middle (MITM)

This happens when someone intercepts the traffic to alter, drop, or send new traffic mimicking the sender. The way they intercept the traffic is by impersonating the server and intercepting the initial handshake and subsequent requests.
A handshake establishes a connection between a client and server.

### Analogy

Building on the lock and key analogy above, we can add Mike, who intercepts requests and impersonates Adam by providing his own lock! This means that any subsequent messages are locked using Mike's lock which enables Mike to read any messages stored in the container.

## Certificates

A certificate is a digitally signed statement that binds an entity's public key to an identity.
*E.g. This public key belongs to google.com.*
This statement is signed by the Certificate Authority (CA) whom the browser trusts.
Browsers trust a list of CAs. These CAs are rigorously audited to ensure upmost security.

### Certificate Authority

An organisation trusted to sign certificates.
Browsers and Operating Systems contain a trust store which contains Root Certificates from CAs. Any certificate chaining to a trusted root certificate is treated as valid.

If a CA is compromised (like Digotor in 2011) or mis-issues certificates, attackers can issue "valid" certificates that enable MITM attacks again. This is why their security is so important.

### Mitigating MITM Attacks

Certificate Authorities ensure that identities are verified against public keys. If you request `example.com` which provides a certificate, your browser will verify this certificate using a root certificate baked into the browser or OS. The root certificate verifies that the `example.com` certificate is valid and was issued the the trusted CA.

MITM attacks can still occur if one of the following occurs:
- CA is corrupted and issues malicious certificates used by attackers.
- A malicious root certificate is installed to the browser or OS.
- Attacker has the requested server's private key (rare).


