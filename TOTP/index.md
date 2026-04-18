# Time-based One-Time Passwords (TOTP)
- [What is TOTP](#what-is-totp)
- [How it works](#how-it-works)
  - [Setup](#setup)
  - [Code generation](#code-generation)
  - [Verification](#verification)
- [Where it is used](#where-it-is-used)
- [Sources](#sources)

## What is TOTP

TOTP (Time-based One-Time Password) is a temporary, single-use passcode generated from a **shared secret key** and the **current time**.
It is defined in **RFC 6238** and built on top of **HOTP** (HMAC-based One-Time Password, RFC 4226).

Unlike a static password, each code is only valid for a short time window (typically **30 seconds**), making it resistant to replay attacks.
TOTP is the algorithm powering authenticator apps and is the most common implementation of the "something you have" factor in [MFA](https://barakadax.github.io/blog?article=MFA).

## How it works

### Setup

1. The server generates a random **secret key** (typically 160 bits, encoded as Base32).
2. The secret is shared with the user's authenticator app, usually via a **QR code** encoding an `otpauth://` URI.
3. Both sides now share the same secret and never exchange it again.

The URI format:
```shell
otpauth://totp/LABEL?secret=BASE32SECRET&issuer=ISSUER&algorithm=SHA1&digits=6&period=30
```

### Code generation

Every 30 seconds, both the app and the server independently compute the same code from the shared secret:

1. **Time step** — `T = floor(unix_time / 30)`
2. **HMAC** — `HMAC-SHA1(secret, T)` → 20 bytes
3. **Dynamic truncation** — use the last byte's low 4 bits as offset `O`, extract 4 bytes starting at `O`
4. **Bit masking** — apply `& 0x7FFFFFFF` to strip the sign bit → 31-bit integer
5. **Code** — `integer mod 10^6`, zero-padded to 6 digits

![TOTP](https://raw.githubusercontent.com/barakadax/barakadax.github.io/refs/heads/master/projImg/TOTP.png)

### Verification

The server performs the same computation using the stored secret and the current time step.
To account for **clock drift** between devices, servers typically accept codes from **±1 window** (the previous and next 30-second slot as well).

> [!NOTE]
> The secret must never leave the server after enrollment. Anyone who obtains the Base32 secret can generate valid codes indefinitely.

## Where it is used

- **Authenticator apps**: Google Authenticator, Microsoft Authenticator, Authy.
- **Web services**: GitHub, Google, AWS, and any service offering 2FA.
- **Infrastructure**: VPN and SSH access via PAM modules.

## Sources

- [Computerphile](https://www.youtube.com/watch?v=ZXFYT-BG2So)
- [Wikipedia](https://en.wikipedia.org/wiki/Time-based_one-time_password)
- [TOTP Python project](https://github.com/barakadax/TOTP)
- [RFC 6238](https://datatracker.ietf.org/doc/html/rfc6238)
- [RFC 4226](https://datatracker.ietf.org/doc/html/rfc4226)
