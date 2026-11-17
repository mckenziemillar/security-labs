# Password Cracking with Hashcat

**Topic:** Hash cracking — brute-force and dictionary attacks against password hashes

## Goal
Recover plaintext passwords from their hashes two ways: a brute-force (mask)
attack against raw MD5 hashes, and a dictionary attack against Linux
`shadow`-file hashes using the `rockyou` wordlist. The point isn't the passwords
themselves — it's understanding *why* some hashes fall in seconds and others
don't.

## Tools
- hashcat (mask and dictionary attacks)
- `rockyou.txt` wordlist
- A Linux environment (WSL / VM / native)

## Brute-force attack (raw MD5)
Three MD5 hashes, passwords of unknown content, length under 10. MD5 is fast and
unsalted, so a mask attack over the full keyspace is viable.

Run one hash at a time and let the mask grow by length:
```bash
echo '<hash>' > hash1.txt
hashcat -m 0 -a 3 -i --increment-min 1 hash1.txt ?a?a?a?a?a?a?a?a?a
hashcat -m 0 hash1.txt --show        # print the cracked result
```
- `-m 0` = raw MD5, `-a 3` = brute-force/mask
- `?a` = the full printable-ASCII set (95 chars), matching "any ASCII character"
- `-i` grows the mask from 1 up to 9, so short passwords are found first

All three cracked at length 5. The reason the mask has to reach length 5 (and
can't stop at 4) is the whole lesson: each extra character multiplies the work
by 95.

## Dictionary attack (Linux shadow hashes)
Linux stores password hashes in `/etc/shadow`, typically as `sha512crypt`
(`$6$...`) — deliberately slow and salted. Brute force is impractical, so you
feed real-world passwords from `rockyou`:
```bash
cut -d: -f1,2 shadow > shadow_hashes.txt      # keep user:hash
hashcat -m 1800 -a 0 --username shadow_hashes.txt rockyou.txt
hashcat -m 1800 shadow_hashes.txt --show
```
- `-m 1800` = sha512crypt, `-a 0` = wordlist mode, `--username` = input has a
  `user:hash` column

Dictionary works here because real users pick guessable passwords — the salt
stops precomputed rainbow tables, but it does nothing against a weak password
that's already in the wordlist.

## Why length beats complexity (keyspace)
Maximum guesses to exhaust a search space is `charset_size ^ length`:

| Requirement | Charset | Keyspace |
|---|---|---|
| 5 chars, digits | 10 | 100,000 (1.0e5) |
| 5 chars, lowercase + digits | 36 | 60,466,176 (6.0e7) |
| 8 chars, lowercase + digits | 36 | 2.82e12 |
| 12 chars, lowercase + digits | 36 | 4.74e18 |
| 5 chars, upper+lower+digits+36 special | 98 | 9.04e9 |
| 8 chars, same 98-char set | 98 | 8.51e15 |

Length is in the **exponent**, charset is in the **base**. Going from 5 → 8
lowercalpha-numeric characters multiplies the keyspace by 36³ ≈ **46,656×**.
Widening the 5-char charset from 36 → 98 only multiplies it by ≈ **150×**. Adding
length buys far more security than adding character variety — which is why
passphrases beat short "complex" passwords.

## Defender takeaways
- Hash with a **slow, salted** algorithm (bcrypt, Argon2, sha512crypt) — never
  raw MD5/SHA-1.
- **Salting** gives each password a unique random value before hashing, so
  identical passwords get different hashes and precomputed rainbow tables are
  useless.
- Push **length** over forced symbol complexity; encourage passphrases.
- Block known-breached passwords (the `rockyou` lesson) and don't reuse
  passwords across sites — one dump then unlocks the rest.
