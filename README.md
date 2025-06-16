# MTF-127 (draft)

**MTF-127** is a working draft of a lightweight, bit-driven compression format optimized for small data blocks. It’s based on a simple *move-to-front* style dictionary logic with minimal overhead and strong adaptability.

🧪 *The project is under development – the format is still taking shape, but the concept is solid.*

---

## 🎯 What It’s About

MTF-127 is:

- **bit-efficient compression** with low overhead  
- a **streamable format** without byte alignment  
- an **adaptive weighted dictionary** – the more frequent a phrase, the higher its position  
- **well-suited for JSON, logs, metadata, short texts, and config files**  
- written in **TypeScript**, aiming for lightweight usage  

---

## 🧠 Output Stream Structure

Output data is written bit by bit, according to the following scheme:

### 🔹 Literal (new character):
```
[1][8-bit char]
```
- Total of 9 bits  
- The inserted character is also added to the dictionary as a new phrase with weight 1

### 🔹 Dictionary reference:
```
[0][7-bit index][8-bit new char]
```
- Total of 16 bits  
- The output inserts the phrase from dictionary[index]  
- A new phrase is formed as: `phrase_from_index + new_char`  
- This gets added to the dictionary (or its weight is increased if it already exists)

---

### 📌 The Dictionary

- Dynamically maintained as a **weighted list**  
- Each time a phrase is used, its **weight increases**, possibly moving it to a higher position  
- Only the **first 127 entries** can be addressed in the bitstream  
- The most frequent phrases get the shortest encoding → compression improves over time

---

## 🚧 Development Status

This repository currently serves as a working draft and idea notebook:

- [x] Bitstream format proposal  
- [x] Weighted dictionary logic  
- [x] `spec.md` with sample examples  
- [ ] Bit encoder (TS)  
- [ ] Decoder and validator  
- [ ] Input data test suite  
- [ ] Benchmarking against gzip, zstd, brotli…

---

## 🛠️ Intended Use Cases

- embedded formats  
- transmission over weak channels  
- archiving of small structured files  
- experimental codec for compact logging  

---

## 📄 License

MIT — use it, remix it, improve it.
---

## 🤖 Author

Dominik Mašek - nikimasek