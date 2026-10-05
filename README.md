# EHAX-project3
Vigenère and AES-128 (ECB) cipher in C, built from scratch
# EHAX Cipher Project — Vigenère + AES-128 (from scratch)
1. Description

This is my submission for the EHAX project, given by the EHAX seniors. I picked **Project 3: Basic Cipher Implementation** — a command-line tool in C that encrypts and decrypts using either the Vigenère cipher or AES-128 (ECB mode), implemented completely from scratch with no crypto libraries.

I'm a first-year CSE student, so this was genuinely my first time going anywhere near real cryptography. I used Claude (Anthropic's AI) heavily to learn the concepts, debug, and understand *why* each piece works, not just to get working code.

2. Features

- Menu-driven CLI — pick a cipher, pick encrypt or decrypt, pick your input source
- Vigenère cipher (classical, keyword-based shifting)
- AES-128 built from scratch — S-box, key expansion, all four round operations, ECB mode
- PKCS#7 padding, so messages of any length work in AES's fixed 16-byte blocks
- Takes input either as a typed message or from a `.txt` file
- AES output/input as hex

3. How I built this

Started out not really understanding what "cipher" meant beyond "secret code." Went through it in this order:

1. **Caesar cipher first** — just to get the basics down. Shift every letter by a fixed number, wrap around using `% 26`. This is where I actually understood ASCII values and why `(x + 26) % 26` is needed to keep shifts positive in C.
2. **Then Vigenère** — same shifting idea as Caesar, but the shift amount comes from a repeating keyword instead of one fixed number. Took me a while to get `key_index % key_len` — that's what makes the keyword loop once it runs out of letters.
3. **Then AES-128**, which is where things got genuinely hard. I went piece by piece:
   - Found out the **S-box** is a fixed 256-value lookup table used to scramble bytes — didn't derive it mathematically (nobody does, even the official AES spec just publishes the table), just learned what it does and used it correctly.
   - Then **key expansion** — took me a few tries to understand why one 16-byte key turns into 11 "round keys" (176 bytes total). Eventually got that it's so each of AES's 10 scrambling rounds uses a related-but-different key instead of reusing the same one.
   - Then the **four round operations** — SubBytes (S-box lookup), ShiftRows (shuffles byte positions), MixColumns (mixes bytes together mathematically), AddRoundKey (XORs with that round's key).
   - Then **PKCS#7 padding** — pads the message to a clean multiple of 16 bytes, where the padding byte's value itself tells decryption how much padding to strip off later.
   - Finally tied it together in **ECB mode** — encrypting every 16-byte block independently.

Along the way I also originally considered building a keylogger (option 4 on the project list) but switched to the cipher project instead.

4. A mistake I made while testing (keeping this honest)

First time I tested decrypt, I typed my original plaintext message back into the decrypt prompt instead of the hex output from encryption — got garbage, since decrypt expects hex input, not plaintext. Good reminder that encrypt and decrypt aren't symmetric in *input format*, even though the underlying math is reversible.

5. Functions

- `vigenere_process()` — does both encrypt and decrypt for Vigenère (decrypt is just encrypt with the shift negated)
- `sbox_substitute()` — looks up one byte in the S-box
- `key_expansion()` — generates all 11 round keys from the original 16-byte key
- `sub_bytes()` / `inv_sub_bytes()` — substitute every byte in the 4x4 state using S-box / inverse S-box
- `shift_rows()` / `inv_shift_rows()` — shuffle byte positions row by row
- `mix_columns()` / `inv_mix_columns()` — mix each column's bytes using AES's finite-field math
- `add_round_key()` — XORs the state with the current round's key
- `aes_encrypt_block()` / `aes_decrypt_block()` — runs one 16-byte block through all 10 rounds
- `aes_process()` — the full pipeline: builds the 16-byte key, pads the message, loops over blocks in ECB mode, handles hex conversion
- `read_file_to_string()` — reads a `.txt` file into the message buffer

5. What I'd add if I had more time

- CBC mode with an IV
