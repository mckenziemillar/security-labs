# RSA Encryption & Primality Testing

**Topic:** Public-key cryptography from scratch — primality testing, RSA key
generation, and file encryption/decryption

**Note:** This was a project for an algorithms course. I'm describing the design
and results here rather than posting the full solution, since it's an active
course assignment. Happy to walk through the implementation on request.

## Goal
Build the core of RSA by hand, in Python, with no crypto libraries:
1. A primality test fast enough to find large random primes
2. RSA key-pair generation (public and private keys)
3. Encrypt and decrypt real files with those keys

## How it works

### Primality testing — Fermat vs. Miller-Rabin
Both tests rest on **modular exponentiation** done by repeated squaring, which is
what keeps them fast on huge numbers:

```python
def mod_exp(x, y, N):      # computes x^y mod N
    result = 1
    x = x % N
    while y > 0:
        if y % 2 == 1:
            result = (result * x) % N
        y = y >> 1
        x = (x * x) % N
    return result
```

**Fermat's test** picks random bases `a` and checks whether `a^(N-1) mod N == 1`.
It's simple but has a real weakness: **Carmichael numbers** (561, 1105, 1729, …)
are composite yet pass Fermat for almost every base, so Fermat can call them
prime.

**Miller-Rabin** fixes that. It writes `N-1 = 2^r · d` with `d` odd, then checks
that the sequence of repeated squares hits the right values. It catches
Carmichael numbers that fool Fermat, and its error probability drops to at most
`4^(-k)` for `k` rounds. I verified my implementation against trial division for
every number up to 20,000 (zero disagreements) and confirmed it correctly rejects
Carmichael numbers that Fermat is designed to miss.

### Key generation
Generate two large random primes `p` and `q`, set `N = p·q` and
`φ(N) = (p-1)(q-1)`, pick a public exponent `e` (I used the standard small values
like 65537), then find the private exponent `d` as the modular inverse of `e`
using the **Extended Euclidean Algorithm**. Public key is `(N, e)`; private key is
`(N, d)`.

### Encrypt / decrypt
A file is read as bytes, split into blocks smaller than `N`, and each block `m`
becomes `c = m^e mod N` (encrypt) or back via `m = c^d mod N` (decrypt) — the same
`mod_exp` doing the heavy lifting. A length header plus padding keeps the original
file size exact on the way back out.

I tested this end to end by exchanging public keys with a classmate: I encrypted a
message with *their* public key, they decrypted it with their private key, and
vice versa — a real two-party key exchange.

## Empirical findings
I measured runtime from 64 up to 2048-bit keys and compared it to the theoretical
Big-O:

- **Key generation:** theory predicts O(n⁴) (the n factor from the Prime Number
  Theorem times O(n³) per Fermat test), but measured growth tracked closer to
  **O(n³)** — in practice a prime shows up faster than the worst-case bound, so the
  per-test cost dominates.
- **Encryption** was far cheaper than decryption, because `e` is a small fixed
  exponent while `d` is a full-size number — so decryption does far more squaring.
  At 2048 bits, decryption took ~140 s vs. ~90 ms to encrypt.

## Security notes
This is **textbook RSA** — good for learning the math, not for production. It has
no padding scheme like OAEP, so it's deterministic (the same message always
encrypts to the same ciphertext) and vulnerable to attacks that real RSA
implementations defend against. The takeaways that *do* carry over: use a
vetted library in practice, pick primality tests that aren't fooled by edge cases
(Miller-Rabin over Fermat), and remember that the private exponent is what makes
decryption expensive.
