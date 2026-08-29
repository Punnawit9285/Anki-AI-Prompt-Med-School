**Ultimate anki flashcard AI prompt for med school students**

https://claude.ai/code/artifact/a400c085-faf1-4c36-8e32-55f2f224ae84?via=auto_preview

**How to use?**

1. Copy this into ANY generative AI ex. Gemini, ChatGPT, Cluade. You will get plain text and .txt file

**--> PROMPT.md :** https://github.com/Punnawit9285/Anki-Prompt-for-Med-School/blob/main/PROMPT.md

! for Claude) recommend using this with Claude code in Vscode (so that it doesn't have token limitation issue)

3. Open Anki --> click "import file" button (at the button of deck page)
**- Card type : Cloze (very important!!)**
<img width="1948" height="2148" alt="image" src="https://github.com/user-attachments/assets/9ebeda81-91e0-48a6-8afe-60536bc795fb" />
!! be careful) select the correct deck

3. Done! (edit as you wish)
**The intention of this flashcard is to be used as pre-lecture template. Furthur information/notes should be reviewed by human.

**Features**

- **Structured Layout & Metadata**
  - Enforces bold tags (`<b>Heading</b>`) for titles and subtopic headers, separated by double line breaks (`<br><br>`).
  - Wraps the `Extra` back-field in `<font color='#55aaff'>`, separating notes from citations with em-dashes (` — `).
  - Assigns single, lowercase tags matching component domains and topic acronyms (e.g., `bsxtranscription`).
    
- **Deterministic Deck Sorting**
  - Prefixes cards with zero-padded slide keys (e.g., `[003]`) to ensure correct sorting in the Anki browser.
  - Maps slide ranges using reversed high-to-low keys (e.g., `[013-012]`) to place summaries immediately after covered content.
  - Appends sequential lowercase letters (`[099a]`, `[099b]`) for multi-card slides to preserve reading order.

- **Information Design & Selective Clozing**
  - Applies the minimum information principle, targeting 2–3 clozes per group (`{{c1::...}}`).
  - Mandates inline underline tags inside cloze boundaries (`{{c1::<u>answer text</u>}}`).
  - Keeps numerical metrics, doses, and measurements as unclozed plain text context.
  - Tracks repeated cloze targets within a subtopic using an external `[sa]` marker (`{{c1::<u>Target</u>}}[sa]`).

- **MathJax & Chemical Formatting**
  - Renders formulas and equations via inline MathJax `\(\mathrm{...}\)`, banning display brackets `\[ \]`.
  - Prevents Anki double-brace parse collisions by placing charge superscripts outside `\mathrm{}` (e.g., `\(\mathrm{HPO_4}^{2-}\)`).
  - Standardizes causal arrows using `-->` in prose and `\rightarrow` in MathJax.

- **CSV Architecture & Data Safety**
  - Standardizes field structure as `"Text";"Extra";"Tags"` with semicolon delimiters.
  - Uses single quotes for inline HTML attributes (e.g., `color='#55aaff'`) to prevent CSV string literal corruption.
  - Forbids internal semicolons and unescaped double quotes inside data fields.
  - Strips automatic LLM bracket citations to maintain clean card text.


- **Automated Validation**
  - Requires a pre-emission check to verify column delimiter counts, MathJax syntax, cloze indexing, and slide ordering before code output.

**What will it looks like?**

Front :
<img width="2032" height="956" alt="image" src="https://github.com/user-attachments/assets/c83a65d1-fdee-452a-b242-ad1b2056a63a" />

Back : <img width="3086" height="1064" alt="image" src="https://github.com/user-attachments/assets/8171955c-d017-42c8-933e-e1388246b195" />


- Alphabetically/numberically order when click on "sort field" column head :
Use 3 digits and greater number come first eg. [013-012] in order to make it sort properly (that's how SQL works) 

- If user wants to add cards/page, I suggest putting [page number] as the first thing so that when sorted it's directly next to existing card with that specific page/pages

<img width="2986" height="2140" alt="image" src="https://github.com/user-attachments/assets/f760bbf2-f217-48a9-a47b-f60098668fa8" />


