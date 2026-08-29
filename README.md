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

CSV Architecture & Data Safety

Uses exact "Text";"Extra";"Tags" field layout with semicolon column delimiters.

Restricts HTML attributes to single quotes (e.g., color='#55aaff') to prevent broken CSV string literals.

Strictly forbids internal semicolons and unescaped double quotes within fields.

Strips LLM auto-citations (e.g., ``) to maintain clean card text.

Deterministic Deck Sorting

Enforces zero-padded slide keys (e.g., [003]) on the first line to guarantee correct Anki browser sorting.

Handles slide ranges via reversed indexing (e.g., [013-012]) to group multi-slide summaries directly after their source slides.

Uses within-slide sequence letters ([099a], [099b]) to retain true top-to-bottom reading order after import.

Information Design & Cloze Rules

Implements minimum information principle with 2–3 target blanks per cloze group ({{c1::...}}).

Mandates underline tags inside cloze boundaries ({{c1::<u>answer text</u>}}).

Leaves numerical values, doses, and measurements as unclozed plain text context.

Uses [sa] markers outside braces for repeated cloze targets within the same subtopic.

MathJax & Chemical Syntax

Enforces inline MathJax \(\mathrm{...}\) for chemical species and equations, banning display math brackets \[ \].

Fixes Anki double-brace parse collisions by placing charge superscripts outside \mathrm blocks (e.g., \(\mathrm{HPO_4}^{2-}\)).

Standardizes causal relationships using --> in prose and \rightarrow in MathJax.

Card Layout & Extra Field

Requires uniform subtopic headers in bold (<b>Subtopic Header</b>) separated by double line breaks (<br><br>).

Formats the Extra field inside <font color='#55aaff'> with supplementary context separated from citations by em dashes (—).

Generates single, lowercase hierarchical tags combining component type and topic (e.g., bsxtranscription).

Quality Assurance

Includes an automated pre-output checklist to verify slide ordering, cloze indices, MathJax syntax, and column delimiters prior to code block emission.

**What will it looks like?**

Front :
<img width="2032" height="956" alt="image" src="https://github.com/user-attachments/assets/c83a65d1-fdee-452a-b242-ad1b2056a63a" />

Back : <img width="3086" height="1064" alt="image" src="https://github.com/user-attachments/assets/8171955c-d017-42c8-933e-e1388246b195" />


- Alphabetically/numberically order when click on "sort field" column head :
Use 3 digits and greater number come first eg. [013-012] in order to make it sort properly (that's how SQL works) 

- If user wants to add cards/page, I suggest putting [page number] as the first thing so that when sorted it's directly next to existing card with that specific page/pages

<img width="2986" height="2140" alt="image" src="https://github.com/user-attachments/assets/f760bbf2-f217-48a9-a47b-f60098668fa8" />


