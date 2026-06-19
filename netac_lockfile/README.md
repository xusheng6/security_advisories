# Netac NetacLockFile — broken file encryption

Two distinct vulnerabilities in **NetacLockFile** (`NetacLockFile.exe` version 1.1.1.2), the software file-encryption utility bundled with Netac encrypted USB flash drives. The tool is marketed as encrypting user files under a user-chosen password, but in the tested version the encryption is broken in two independent ways. Each is tracked as a separate CVE.

| Advisory | Defect | CWE | CVSS v3.1 |
|---|---|---|---|
| [partial-encryption.md](partial-encryption.md) | Only ~1/3 of the file body is encrypted; ~2/3 is stored in cleartext | [CWE-311](https://cwe.mitre.org/data/definitions/311.html) | 5.4 (Medium) |
| [hardcoded-key.md](hardcoded-key.md) | The encrypted portion uses a fixed AES key hardcoded in the binary, independent of the user password | [CWE-321](https://cwe.mitre.org/data/definitions/321.html) | 5.4 (Medium) |

The two defects are independently fixable and have different root causes, which is why two CVE IDs are requested. Together they allow complete recovery of the original plaintext from any container file the tool produces.

Both findings are presented in the reporter's talk *Breaking Encrypted USB Drives* at [RE//verse 2026](https://re-verse.io/) (March 5–7, 2026, Orlando, FL). Recording: <https://www.youtube.com/watch?v=Rv6jdnQ4YhY>.

Discovered and reported by Xusheng Li, in a personal capacity. Reported to Netac on 2025-10-19; no vendor response or fix as of publication.
