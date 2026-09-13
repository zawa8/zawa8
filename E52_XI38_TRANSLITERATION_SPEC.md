# ADAS Font Rendering & Transliteration Specification
**Target System:** Next.js Rule-Based Synchronous Transliteration Engine  
**Font Mapping Standard:** `e52` → `xi38` / `htrlib` (`hindixv38.ttf`)

---

## 1. Visual Reading (दृष्टिगत पठन और संदर्भ)
Visual reading in the `xi38` engine relies on human cognitive context recognition rather than 1:1 literal symbol matching. By maintaining standard ASCII tokens for input data integrity, the system leverages human reading patterns to resolve phonetic ambiguities on UI outputs.

* **Context-Driven Perception:** Readers automatically infer the intended Devanagari word based on sentence context even when glyphs share visual characteristics (e.g., reading `jrbuz` as *Tarbuz* or `vnuman` as *Hanuman*).
* **ASCII Data Integrity:** Backend search indexes, voice-to-text engines, and database records preserve pristine ASCII tokens (`T`, `k`, `v`), ensuring searchability while the font handles rendering nuances.

---

## 2. Dual-Glyph Hacks (हाइब्रिड ग्लिफ़ डिज़ाइन)
To maximize readability and backward compatibility without expanding character slots, custom hybrid glyphs are modified directly inside `hindixv38.ttf` using Retrofit Glyph design techniques:

| Base Slot | Structural Modification | Visual Duality | Primary / Secondary Sound | Target Example |
| :--- | :--- | :--- | :--- | :--- |
| **`J`** | Added subtle downward arrow to stroke | **J / T** | `J` (ज) / `T` (त) | `Jrbuz` (तरबूज) |
| **`Q`** | Integrated inner `D` loop within outer ring | **Q / D** | `Q` (क/क्व) / `D` (द/द्व) | `Qwai` / `Dwai` (द्वई) |
| **`V`** | Added horizontal crossbar in middle ($\forall$) | **V / H** | `V` (व) / `H` (ह) | `vnuman` (हनुमान) |
| **`X`** | Added bottom horizontal base bar | **X / A** | `X` (श/ष) / `A` (अ/आ/मात्रा) | `xnar` (अनार), `xam` (आम) |

---

## 3. Phonetic Distinction & Consonant Mapping
Clear boundaries are established between Dental (दंत्य) and Retroflex (मूर्धन्य) consonants to avoid rendering collisions in ADAS displays:

* **Dental Consonants (त-वर्ग):**
  * `T` / `t` $\rightarrow$ **त** (`ta` as in *Trbuz*)
  * `Th` / `th` $\rightarrow$ **थ** (`tha` as in *Thali*)
* **Retroflex Consonants (ट-वर्ग):**
  * Standard ASCII slots handle **ट** (`Ta`) and **ठ** (`Tha`) (as in *tmatr*, *thnda*).
* **Sibilants & Vowels:**
  * `x` mapped via Hybrid `X/A` to serve initial vowel roots (*Anar*) and medial long vowels (*Aam*, *Sagar*).

---

## 4. Deterministic Token Logic (`kar` vs `kr` vs `kyet`)
To ensure high-performance synchronous execution in Next.js, the pipeline eliminates intermediate translation APIs in favor of strict deterministic token rules:


* **`kr`** $\rightarrow$ mapped strictly to **कर** (Action verb / *kaam kr*).
* **`kar`** $\rightarrow$ mapped strictly to **कार** (Vehicle noun / *maruTi kar*).
* **`kyet`** $\rightarrow$ natural phonetic representation for **कैट** (*kyet billi*), entirely replacing ambiguous fallback tokens like `cxr` or `cxt`.

---

## 5. Architectural Recommendations for Next.js Pipeline
1. **Font Forge Isolation:** Keep `J`, `Q`, `V`, `X` hybrid outlines strictly in the TTF renderer layer. Do not mutate underlying text strings in state.
2. **Synchronous Execution:** Implement mapping as a single-pass lookup table (O(1) complexity per token) to meet low-latency safety requirements in automotive font rendering.
3. **Fallback Safety:** Maintain strict standard ASCII tokens (`fiwe`, `wiolet`) alongside dual glyphs for zero-recall-failure safety compliance.