# Hashing vs Encryption

Encryption is reversible with a key. Hashing is not reversible by design. They solve different problems.

## Comparison

| | Hashing | Encryption |
|---|---|---|
| Direction | one-way | two-way (encrypt / decrypt) |
| Key | none (except keyed variants like HMAC) | required |
| Output size | fixed | grows with input |
| Goal | integrity, fingerprints, password storage | confidentiality |
| Example | SHA-256, bcrypt, argon2 | AES, RSA, ChaCha20 |

Related but different: **encoding** (Base64, URL encoding) is just representation. It provides no secrecy at all.

## Hashing

```js
const crypto = require("crypto");

crypto.createHash("sha256").update("hello").digest("hex");
// 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
```

Properties of a good cryptographic hash:

- Same input, same output.
- A small change in input changes the output completely.
- You can't practically recover the input from the output.
- Finding two inputs with the same hash (collision) is infeasible. MD5 and SHA-1 no longer meet this, so don't use them for security purposes.

Uses: file integrity checks, content addressing (Git uses hashes), digital signatures (sign the hash), password storage.

## Passwords are special

General-purpose hashes like SHA-256 are designed to be fast. That's bad for passwords, because attackers can try billions of guesses per second on stolen hashes.

Password hashing uses deliberately slow, tunable algorithms: **argon2**, **scrypt**, **bcrypt**, or PBKDF2. They also include a **salt**: a random value stored alongside the hash.

A salt means:

- Two users with the same password get different hashes.
- Precomputed tables (rainbow tables) don't work.

The salt is not secret. It just needs to be unique and random.

```js
const crypto = require("crypto");

function hashPassword(password) {
  const salt = crypto.randomBytes(16);
  const hash = crypto.scryptSync(password, salt, 64);
  return `${salt.toString("hex")}:${hash.toString("hex")}`;
}

function verifyPassword(password, stored) {
  const [saltHex, hashHex] = stored.split(":");
  const expected = Buffer.from(hashHex, "hex");
  const actual = crypto.scryptSync(password, Buffer.from(saltHex, "hex"), 64);
  return crypto.timingSafeEqual(expected, actual);
}
```

This is a learning example using Node's built-in `scrypt`. In a real app, use a maintained library or framework helper for argon2/bcrypt with sensible parameters, and use the async versions so you don't block the [event loop](../../programming/javascript/event-loop.md).

## Encryption

- **Symmetric** (AES, ChaCha20): one shared key encrypts and decrypts. Fast. Problem: how to share the key.
- **Asymmetric** (RSA, elliptic-curve): a public key encrypts (or verifies), a private key decrypts (or signs). Slower. Solves key distribution.

In practice they're combined. TLS uses asymmetric cryptography during the handshake to agree on keys, then symmetric encryption for the actual traffic.

Use authenticated encryption modes (e.g. AES-GCM) so tampering is detected, not only hidden.

## Common mistakes

- "Encrypting" passwords. If you can decrypt them, so can an attacker who gets the key. Hash them.
- Using SHA-256 alone for passwords.
- Rolling your own crypto or inventing a scheme by combining primitives.
- Treating Base64 as protection.
- Hardcoding keys in source code or committing them.
- Reusing a nonce/IV where the algorithm forbids it.
- Comparing hashes or tokens with `===`, which can leak timing information. Use a constant-time comparison.

## Practical notes

- Need to check integrity with a secret? Use HMAC, not a plain hash with a secret glued on.
- JWT signatures are HMACs or public-key signatures over the token content. See [JWT vs sessions](../../backend/authentication/jwt-vs-sessions.md).
- Parameters for password hashing (cost factors) change over time. Follow current OWASP guidance.

## Remember

- Hash to verify, encrypt to hide.
- Passwords: slow hash + salt.
- Don't invent crypto.

## Related

- [Authentication vs authorization](../../backend/authentication/authentication-vs-authorization.md)

## References

- OWASP Password Storage Cheat Sheet
- Node.js docs: `crypto` module
