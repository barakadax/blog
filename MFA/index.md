# Multi Factor Authentication (MFA)
- [What it is](#what-it-is)
- [Why you need it](#why-you-need-it)
- [The types of authentication](#the-types-of-authentication)
- [Best practices](#best-practices)
- [Sources](#sources)

## What it is

MFA is an access control method that requires two or more independent forms of evidence (factors) to verify a user's identity. By combining factors from different categories — typically "something you know", "something you have", and "something you are" — MFA raises confidence that the person authenticating is who they claim to be.

## Why you need it

- **Reduces account compromise:** Even if a password is stolen or guessed, additional factors stop many attackers.
- **Mitigates common attacks:** Protects against credential stuffing, phishing, keyloggers, and brute-force attacks.
- **Limits impact of leaked passwords:** With MFA, leaked credentials alone are usually insufficient to gain access.
- **Compliance and risk management:** Many regulations and security frameworks expect or require MFA for privileged access and remote login.
- **Protects sensitive operations:** Set-up MFA for transactions or admin actions reduces fraud and misuse.

## The types of authentication

- **Something you know (knowledge):** Passwords, PINs, or answers to secret questions. Easy to deploy but prone to theft and reuse; should not be the sole assurance for sensitive access.
- **Something you have (possession):** Hardware security keys, smartcards, authenticator apps, or push notifications to a registered device.
- **Something you are (inherence):** Biometrics such as fingerprints, facial recognition, or iris scans. Convenient but carries privacy, recovery, and false‑positive/negative considerations.
- **Other/contextual factors:** Location (IP/geofence), device posture, behavior/typing patterns, and time-of-day can be used for risk-based or adaptive authentication.

## Best practices

While it's beneficial to make multiple authentication methods available so users can choose what fits them, user experience and adoption suffer if systems demand many factors every time. In practice, requiring two distinct factors (2FA) is the best balance between security and usability for most accounts.

- **Prefer resistant methods:** Use phishing-resistant methods (Something you have) where feasible.
- **Avoid SMS when possible:** SMS is better than nothing but vulnerable to SIM swaps and interception risks.
- **Provide recovery:** Give single-use recovery codes and clear recovery steps; encourage secure storage of recovery codes.

## Sources
- [Tom Scott](https://www.youtube.com/watch?v=hGRii5f_uSc)
- [Computerphile](https://www.youtube.com/watch?v=ZXFYT-BG2So)
