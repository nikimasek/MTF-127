# MTF-127 – Bit-Driven Compression Format (Specification Draft)

Version: 0.1 (draft)  
Author: Dominik Mašek – nikimasek  
Last updated: *(to be added later)*

---

## 🧭 Overview

**MTF-127** is an adaptive compression format optimized for small to medium-sized data blocks. It uses a dynamic dictionary with weight-based phrase ordering and a bit-efficient output. This makes it highly compressive with minimal overhead — ideal for structured text, logs, configurations, or embedded transmissions.

---

## 📐 Output Bit Structure

Each entry in the data stream begins with **1 bit**, which determines the type of record:

### Literal (new character)

| Bits | Description           |
|------|------------------------|
| 1    | Type flag = 1 (literal) |
| 8    | Character (8 bits)     |

→ Total of **9 bits**  
→ Character is directly written and also added to the dictionary as a new phrase

### Dictionary Reference

| Bits | Description                    |
|------|--------------------------------|
| 0    | Type flag = 0 (reference)      |
| 7    | Dictionary index (0–126)       |
| 8    | New character (8 bits)         |

→ Total of **16 bits**  
→ Phrase = `dictionary[index] + character`  
→ The resulting phrase is added to the dictionary (or its weight is increased)

---

## 📚 Dictionary

- The dictionary is dynamic, composed of phrases: either a single character or a previous phrase + character  
- Each phrase is assigned a **weight** (integer, initially = 1)  
- On each use, the weight is increased: `weight += 1`  
- After each update, the phrase list is **sorted in descending order by weight**  
- **Indices 0–126** refer to the top 127 phrases by current ranking  
- Entries outside the TOP127 can’t be referenced — only used as building blocks

---

## 📌 Encoder Behavior

1. If a phrase `(something) + character` exists, locate its position in the dictionary:  
   - If it’s in the TOP127 → write as `[0][index][character]`  
   - If not → write the character as a literal `[1][character]`

2. Add the resulting phrase to the dictionary:  
   - If it doesn't exist → add it with weight = 1  
   - If it exists → increase weight, re-sort list

3. Continue with the remaining input

---

## 📤 Output Stream

Entries are written bit by bit without byte alignment.  
During decoding, the parser must read the first bit `b0` to determine record length (9 or 16 bits).

---

## ⚠️ Limitations & Notes

- Currently, phrases outside the TOP127 are not addressable  
- Format is optimized for single-byte characters (ASCII) but can be extended  
- Planned support includes escape codes or extended literals for Unicode

---

## 🔚 Conclusion

This specification is still open and subject to change. If you have ideas or feedback, open an issue or start a discussion. Compression should be both smart and small — and here, it’s being tried bit by bit.