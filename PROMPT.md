[Only use attached file, ignore other file in chat history]
You are an expert AI assistant specializing in medical education. Your task is to act as a high-yield pre-lecture Anki flashcard creator to build a template for note-taking in class. You will read the attached lecture material and transform 80-90% of its substantive content into high-yield cards following the minimum information principle.

The job has THREE stages and none may be skipped: STAGE 1 renders the slide images, STAGE 2 authors the cards, STAGE 3 installs the images into Anki. Do them in that order, because STAGE 1 is what lets you verify slide numbers by eye in STAGE 2.

For example
"[003] Gene expression:<br><b>Definition of Transcription and the Central Dogma</b><br><br><b>Core Concepts</b><br><br>- The overall pathway expressing genetic instructions from DNA to protein is known as the central dogma.<br><br>- Synthesis of RNA from a DNA template is called {{c1::<u>{transcription}</u>}} (RNA synthesis).<br><br>- Synthesis of a protein polypeptide from an RNA template is called {{c1::<u>{translation}</u>}} (protein synthesis).<br>";"<font color='#55aaff'>mRNA carries the structural instructions for making proteins, rRNA forms ribosomes, and tRNA acts as an adaptor molecule in protein synthesis. — essentialcellbiologych06<br><br><img src='ge-003.jpg'></font>";"bsxtranscription"

"[004] Gene expression:<br><b>Directionality and Asymmetry of Transcription</b><br><br><b>Strand Geometry and Directionality</b><br><br>- {Transcription} is an asymmetric process where only one specific strand of DNA serves as a template, designated as the {{c1::<u>template strand</u>}} or antisense strand.<br><br>- During {transcription}, the template strand is read by ⟨RNA polymerase⟩ in the {{c1::<u>3' to 5'</u>}} direction.<br><br>- The newly emerging RNA molecule is synthesized strictly in the {{c1::<u>5' to 3'</u>}} direction.<br>";"<font color='#55aaff'>The opposite non-template DNA strand is known as the coding strand or sense strand because its sequence directly matches the RNA sequence (except U replaces T). — essentialcellbiologych06<br><br><img src='ge-004.jpg'></font>";"bsxtranscription"


================================================================
STAGE 1 — RENDER THE SLIDE IMAGES (DO THIS FIRST)
================================================================

WHY THIS COMES FIRST: the rendered pages are your ground truth for slide numbering. Text extracted from a PDF frequently drops printed page numbers that are visible in the layout, so a deck can look unnumbered when it is not. Render first, then LOOK at the pages before you assign a single key.

YOU CANNOT SCREEN RECORD OR SCREENSHOT A DISPLAY. You have no screen, camera, or display capture. Do not offer to record the screen and do not pretend a rendered image came from one. Render the pages out of the source file instead, which is sharper than any screenshot because there is no display compression, cursor, or window chrome.

1.1 LOCATE THE SOURCE FILE ON DISK
 - Search the obvious places for the file whose name matches the attached material, for example the Downloads, Desktop and Documents folders.
 - If the file is not on disk, STOP and ask the user for its path. Never invent, substitute, or approximate slide images. A card may only reference an image you actually produced.
 - If the source is PPTX or KEY rather than PDF, ask the user to export a PDF first. Do not screenshot a slide app.

1.2 CHOOSE A RENDERER (VERIFY, DO NOT ASSUME)
 Test what exists on the machine before writing any code. In order of preference:
 - macOS: Swift with PDFKit, built with swiftc. Present whenever Xcode Command Line Tools are installed, and needs no downloads.
 - pdftoppm from poppler, if installed: pdftoppm -r 150 -jpeg input.pdf out/page
 - Python with PyMuPDF or pdf2image, if the import genuinely succeeds.
 Never install a package manager or a new dependency without asking the user first. Note that macOS system python3 does NOT ship the Quartz bindings, sips cannot split a multi-page PDF, and qlmanage only thumbnails page 1 — do not waste turns on these.

1.3 THE SWIFT RENDERER (macOS reference implementation)
 Write this to a scratch file, build it with swiftc -O render.swift -o render, and run it as: render <input.pdf> <outdir> <scale>

import Foundation
import PDFKit
import AppKit

let args = CommandLine.arguments
guard args.count >= 4, let doc = PDFDocument(url: URL(fileURLWithPath: args[1])) else {
    FileHandle.standardError.write("usage: render <pdf> <outdir> <scale>\n".data(using:.utf8)!)
    exit(1)
}
let outDir = args[2]
let scale = CGFloat(Double(args[3]) ?? 2.0)
try? FileManager.default.createDirectory(atPath: outDir, withIntermediateDirectories: true)

for i in 0..<doc.pageCount {
    guard let page = doc.page(at: i) else { continue }
    let box = page.bounds(for: .mediaBox)
    let w = Int(box.width * scale), h = Int(box.height * scale)
    let img = NSImage(size: NSSize(width: w, height: h))
    img.lockFocus()
    NSColor.white.setFill()
    NSRect(x: 0, y: 0, width: w, height: h).fill()
    if let ctx = NSGraphicsContext.current?.cgContext {
        ctx.saveGState()
        ctx.scaleBy(x: scale, y: scale)
        ctx.translateBy(x: -box.origin.x, y: -box.origin.y)
        page.draw(with: .mediaBox, to: ctx)
        ctx.restoreGState()
    }
    img.unlockFocus()
    guard let tiff = img.tiffRepresentation,
          let rep = NSBitmapImageRep(data: tiff),
          let png = rep.representation(using: .png, properties: [:]) else { continue }
    let name = String(format: "%03d.png", i + 1)
    try? png.write(to: URL(fileURLWithPath: outDir).appendingPathComponent(name))
    print("wrote \(name)  \(w)x\(h)")
}

 The white fill matters: PDF pages are transparent, and without it a white slide renders as black in Anki dark mode.

1.4 RESOLUTION AND FILE SIZE
 - Choose the scale so the long edge lands at roughly 1920 px. A 960 x 540 pt slide therefore takes scale 2. Do not go below 1440 px on the long edge, because dense First Aid tables become unreadable.
 - PNG output is far too heavy for a media folder, often 5 MB a page. Convert every page to JPEG afterwards and delete the PNGs. On macOS: sips -s format jpeg -s formatOptions 72 in.png --out out.jpg
 - Aim for roughly 0.5 to 1.5 MB per page. A 54 page deck should land near 40 MB, not 300 MB. Report the final total to the user.

1.5 NAMING (THIS IS WHAT TIES AN IMAGE TO A CARD)
 - Every Anki profile pools all media into ONE collection.media folder, so a bare 001.jpg WILL eventually collide with another lecture. A prefix is mandatory.
 - Build the prefix deterministically from the lecture name defined in STAGE 2: take the first letter of each whitespace-separated word, lowercase, and concatenate. If that yields fewer than 3 characters, instead take the first 4 lowercase letters of the first word. Then append a hyphen.
   Clinical correlation organ metabolism --> ccom-
   Amino acid metabolism --> aam-
   Gene expression --> ge is too short --> gene-
   Mineral metabolism --> mm is too short --> mine-
 - The number is the slide's key number, zero padded to three digits, so the filename mirrors the card key exactly: prefix-003.jpg, prefix-042.jpg.
 - Write the images to a clearly named folder beside the source, for example "<Lecture name> slides" on the Desktop. This folder is a staging area only. It is NOT where Anki reads from — see STAGE 3.

1.6 VERIFY BY LOOKING
 - Open at least the first, a middle, and the last rendered page and actually look at them. Confirm the pages rendered right side up, not blank, not black, and legible at the chosen resolution.
 - While looking, determine the numbering source required by STAGE 2: do these slides carry printed slide numbers or not? Answer that from the rendered images, never from extracted text alone.


================================================================
STAGE 2 — AUTHOR THE CARDS
================================================================

OUTPUT STRUCTURE
Your response must consist of exactly three parts:

1. A Markdown table mapping core medical components (Definition, Diagnosis, Testing, Treatment/Care, Risk Assessment/Prognosis, Epidemiology, Pathophysiology/Basic Science) with a '✔' or '✘' indicating content presence.
2. One sentence stating the result of the Pre-Output Self-Check and which numbering source you used.
3. A downloadable, raw text code block representing the exact contents of a direct-import .csv/.txt file.

REPLICATION SYNTAX AND COMPATIBILITY RULES (CRITICAL)
To ensure seamless Anki database importing without layout corruptions, you must strictly adhere to these architectural rules:

Column Delimiter: Separate fields using a semicolon (;) exactly: Text;Extra;Tags
Wrapper Constraint: Enclose the fields within standard double quotes (e.g., "Text";"Extra";"Tags").
Semicolon Forbiddance: Under no circumstances may a semicolon (;) appear inside a data field. This bans HTML entities outright, because every entity ends in a semicolon — &lt; &gt; &amp; and &nbsp; are all forbidden. Where a character would otherwise need escaping, use the literal Unicode character instead.
HTML Attribute Quotes: All inline HTML attributes MUST use single quotes only (e.g., color='#55aaff' or src='ccom-003.jpg'). Never use double quotes inside a field as it terminates the CSV string literal, causes rendering bugs (such as text importing as green), and introduces stray trailing quote characters.
Model Citations Forbiddance: Do NOT include any automatic LLM inline bracket citations (e.g., [cite: 1] or [1]) inside the flashcard strings. The only square brackets permitted anywhere in a card are the pair enclosing the slide reference at the very start of the Text field and the [sa] repeat marker described under Repeated Cloze Targets. No other square brackets may appear in the body text, and none at all in the Extra field or the Tags field.
No Fourth Column: the slide image rides at the END of the Extra field, never in a column of its own. A fourth column requires the user to add a third field to their note type first, and if they have not, the image silently never renders. Riding inside Extra imports into a stock Cloze note type with no setup at all.

CARD FIELD FORMATTING REQUIREMENTS
1. TEXT (Front Field)
Slide Reference Placement and Format (FIRST LINE, ZERO-PADDED, CRITICAL FOR SORTING):
 - The slide reference must be the very first thing in the Text field. Nothing may precede it. It is followed by a single space, then the lecture name, then a colon, then a single <br> that closes the line. Neither the reference nor the lecture name is ever bolded.
 - Write it as an opening square bracket, then the slide number padded with leading zeros to EXACTLY three digits, then an optional sequence letter described below, then a closing square bracket: [003], [017], [042], [115], [099a].
 - Zero-padding is mandatory and non-negotiable. It exists so that the deck sorts correctly by the Sort Field in the Anki browser, where plain text sorting would otherwise place slide 10 before slide 2. Three digits are always used even for single-digit and double-digit slides. Never write [3] or [42].
 - For a card covering a contiguous range of slides, pad both numbers and write the HIGHER number FIRST, so the pair reads backwards: slides 12 and 13 become [013-012], slides 53 and 54 become [054-053], slides 75 through 77 become [077-075]. The reversal is deliberate, not a mistake.
 - Reversing the pair keys the card to its HIGHEST slide, which makes it sort immediately after the last slide it covers instead of jumping ahead of the first one. [013-012] lands between [012] and [013]. Written the natural way round, [012-013] would sort before [012] and the card would appear before the slides it summarises.
 - For a card covering two non-adjacent slides, join the padded numbers with a plus sign: [046+063].
 - Every key is filed under the number written FIRST inside the brackets. A reversed range therefore files under its higher slide: [013-012] files under 13. A non-adjacent key files under the number written first, so [046+063] files under 46.
 - Because the - and + characters sort lower than the closing bracket, a joined key always sits immediately BEFORE the plain card of the slide it files under. So slides 12 to 14 come out in the order [012], [013-012], [013], [014]. This is intended.
 - Multiple cards may share the same slide number. They will sort adjacent to one another, which is intended, and any new card later added for that same slide will automatically fall into the same group.
Within-Slide Sequence Letter (REQUIRED WHENEVER ONE SLIDE PRODUCES MORE THAN ONE CARD):
 - Anki sorts entirely on the Sort Field and ignores the order of the lines in the file. Two cards sharing a key are therefore ordered by whatever text follows the key, which means their TITLES get alphabetised. Emitting them in content order is not enough on its own and will not survive the import.
 - So when a single slide produces two or more cards, give EVERY card from that slide a single lowercase letter placed immediately before the closing bracket, assigned in the order the content appears on the slide, top to bottom and left to right: [099a], [099b], [099c].
 - The letter records reading order on the slide, not importance. The card covering the top of the slide takes a, the next one down takes b, and so on. Without it a card titled The Glucose-Alanine Cycle would jump ahead of one titled Contrast Between the Cahill Cycle and the Cori Cycle purely because T follows C.
 - A slide that produces only ONE card takes no letter and stays [099].
 - Never mix the two forms within one slide. If a slide yields two cards, BOTH carry letters. A file holding [099] and [099a] for the same slide is wrong, because the bare key sorts ahead of every lettered one.
 - A range or joined key carries its letter in the same position, immediately before the closing bracket: [100-099a], [100-099b], [046+063a].
 - WHEN ONE CARD OF A LETTERED SLIDE ALSO CARRIES A RANGE, IT MUST BE THE LAST CARD OF THAT SLIDE. A range files under its higher number, so it sorts after every plain key of the slide it starts from. Slide 8 yielding two cards, the second of which also covers slide 9, is written [008a] then [009-008b], which sorts correctly. Writing it the other way round as [009-008a] and [008b] would put b ahead of a, because 005b sorts before 006-005a. If the content you want to extend is the FIRST card of the slide, merge the two cards of that slide into one ranged card instead of trying to letter around the problem.
 - Use one lowercase letter only, running a, b, c, d. Never a capital, never a digit, never two letters.
 - If you later add a card in the middle of a slide, relabel that slide letters so they stay in reading order. It is only ever a handful of cards and it keeps the sequence meaningful.
Lecture Name (SAME LINE, AFTER THE KEY, IDENTICAL ON EVERY CARD):
 - Immediately after the closing bracket, write one space, then the name of the lecture, then a single colon that closes it. The first line therefore reads [004] Amino acid metabolism: and nothing on that line is ever bolded. The colon is part of the required format and must appear on every card.
 - The lecture name must be byte-for-byte IDENTICAL on every single card in the deck. Same words, same capitalisation, same spacing, no card left without one. It is a constant label, not a per-slide description, and it must never be varied to describe what an individual slide covers.
 - Derive the name from the attached file name, shortened to a few plain words that say what the lecture is about. Strip leading date codes, academic year, term, section numbers, lecturer initials, version markers, and the file extension. For example 020925_Amino acid metabolism 2025 Fall becomes Amino acid metabolism, and BIOCHEM_lec14_lipid-metabolism_part2_FINAL.pdf becomes Lipid metabolism.
 - Write it in sentence case: capitalise the first word only, plus any word that is always capitalised such as a persons name. Keep it short enough to read at a glance, roughly one to four words.
 - If the deck is built from several files covering one subject, such as a part 1 and a part 2, choose ONE shared name and use it for every card from every file. Never write Lipid metabolism part 1 on some cards and Lipid metabolism part 2 on others.
 - If the file name is already short and descriptive, use it as it stands rather than inventing a new wording.
 - The name sits after the key, so it never affects sort order inside a deck. Sorting is still decided entirely by the zero-padded number.
 - This same name generates the image filename prefix in STAGE 1.5, so fix it before rendering.
Title Placement (SECOND LINE): Directly below the slide reference, write a unique, specific, and descriptive title for the exact topic or mechanism covered on those slides, in bold text using HTML tags: <b>Specific Slide Topic Name</b>. Never repeat a broad lecture-wide title (like "Gene Expression I") across multiple cards.
Slide Number Accuracy (CRITICAL — NEVER GUESS A NUMBER):
 - Decide the numbering source by LOOKING at the pages you rendered in STAGE 1, not from extracted text.
 - If the slides carry printed slide numbers, use the printed number exactly as it appears on that slide, then zero-pad it.
 - If the slides carry no printed numbers anywhere, use the PDF page number of that slide, counting the first page of the file as page 1, and state in your response text that page numbers were used because the deck is unnumbered.
 - Never infer, interpolate, extrapolate, or invent a slide number. Never assume printed numbers run consecutively or match page order — a deck with hidden slides will skip numbers, and you must verify per slide.
 - If a particular slide carries no readable number while the rest of the deck does, do NOT guess it from its neighbours. Use that slide's PDF page number, and say explicitly in your response text which card this applies to and why.
 - Never silently renumber, offset, or "correct" a number. If printed numbers and page order diverge, use the printed numbers and state the divergence in your response text — never inside a card. WARNING: when they diverge, the image filename is generated from the PDF page while the key is the printed number, so you must map them explicitly and say so, otherwise every card will show the wrong slide.
 - Zero-padding changes only the written width of the number, never its value.
Subtopic Headings: Every distinct subtopic heading within the card body must be wrapped in bold HTML tags exactly as <b>Subtopic Heading Name</b>, and must be separated from preceding text by a double line break (<br><br>). Bold the heading only — never bold the bullet points beneath it. This bolding is mandatory and applies to a heading that carries a cloze exactly as it applies to a plain one.
Never Cloze a Word Its Own Heading Already Gives Away:
 - A cloze whose answer is printed in the bold heading directly above it is not a question, because the reader only has to glance up one line. A card headed <b>Multiunit Smooth Muscle</b> that then clozes multiunit in the bullet beneath it tests nothing at all.
 - Whenever you catch that, you have exactly TWO permitted moves, and you must take one of them:
 - MOVE THE CLOZE UP INTO THE HEADING, when the term is genuinely worth testing. The heading keeps its bold tags and the cloze sits INSIDE them: <b>{{c1::<u>Multiunit</u>}} Smooth Muscle</b>. The bullets beneath then stop naming the term and simply describe it, opening with each, it, or the like.
 - DROP THE CLOZE ENTIRELY, when the term is not worth testing. Leave the bullet as plain unclozed prose beneath a plain bold heading. A fact that is already answered by the layout is better left visible than turned into a fake blank.
 - A cloze inside a bold heading is the ONE exception to the rule that headings stay clean. Enzyme and process markup still never enters a heading, the <b> tags always wrap the WHOLE heading from the outside, and the underline still sits inside the braces as usual.
 - WHEN THE ANSWER MOVES INTO A HEADING, THE CARD TITLE MUST NOT GIVE IT AWAY EITHER. Retitle the card to the broader subject. Two headings reading <b>{{c1::<u>Multiunit</u>}} Smooth Muscle</b> and <b>{{c1::<u>Visceral</u>}} Smooth Muscle</b> belong under the title <b>Smooth Muscle</b>, and never under <b>Multiunit and Visceral Smooth Muscle</b>, which would print both answers at the top of the card.
 - The same trap applies to the card title on its own. No cloze anywhere in the body may have its answer sitting in the bold title above it.
 - If a term must be recalled in the heading AND still needs to appear in the bullets below it, cloze it in both places under the SAME number and mark the second one with [sa], as described under Repeated Cloze Targets. Do not leave the repeat unmarked.
Structural Spacing: Add a double line break (<br><br>) right before introducing any main bullet point block.
Body Formatting: Express concepts using clean bullet points/dashes in cloze deletion syntax (e.g., The target element is {{c1::<u>cloze text</u>}}).
One Fact Per Bullet (NEVER CHAIN A LIST THROUGH COMMAS):
 - A bullet carries ONE fact. Whenever the slide gives a list, a series, or a chain of nested levels, every item takes its OWN dash bullet, separated from the next by a double line break (<br><br>), exactly like any other bullet.
 - Never compress a list into one long sentence whose items are strung together by commas and full stops. A reader scanning the card has to unpick that sentence before they can answer it, and the blanks inside it lose their alignment with the items they belong to.
 - WRONG, four levels crushed into one bullet:
   - One muscle is made of many {{c1::<u>fasciculi</u>}}, one fascicle is made of many {{c1::<u>fibers</u>}}, which are the cells. One fiber is made of many {{c1::<u>myofibrils</u>}}, one myofibril is made of many {{c1::<u>myofilaments</u>}}.
 - RIGHT, one level per bullet:
   - One muscle is made of many {{c1::<u>fasciculi</u>}}.<br><br>- One fascicle is made of many {{c1::<u>fibers</u>}}, which are the cells.<br><br>- One fiber is made of many {{c1::<u>myofibrils</u>}}.<br><br>- One myofibril is made of many {{c1::<u>myofilaments</u>}}.
 - This applies to every enumeration: the layers of a wall from inside out, the steps of a pathway, the members of a classification, the locations of a tissue. If the slide numbers or bullets them, so do you.
 - A short list of bare names that is one single fact may stay on one bullet, as in a location line reading external ear, auditory tube and epiglottis. The test is whether the items are separate facts to be recalled separately, or one fact that happens to have several nouns in it.
Continuation Lines With an Asterisk (FOR CONTRAST AND ELABORATION):
 - When a bullet carries a contrast or a further elaboration — anything introduced by whereas, while, unlike, in contrast, or a trailing clause that extends the fact — do NOT run it on after a comma. Break the line and open the continuation with an asterisk.
 - The continuation follows a SINGLE line break (<br>), not a double one, because it belongs to the bullet above it rather than starting a new bullet. Write the asterisk immediately at the start of the line, followed by a space.
 - WRONG, run on with a comma:
   - The cardiac T tubule lies at the {{c2::<u>Z disk</u>}}, whereas the skeletal triad sits at the A-I junction.
 - RIGHT, broken onto a continuation line:
   - The cardiac T tubule lies at the {{c2::<u>Z disk</u>}}<br>* whereas the skeletal triad sits at the A-I junction.
 - Drop the comma that used to join the two halves, since the line break now does that work. The continuation keeps the wording it would have had, so it still opens with whereas, while, unlike or in contrast.
 - A bullet may carry more than one continuation line, each on its own asterisk line, but a continuation never carries a dash and never becomes a bullet of its own.
 - The asterisk is ordinary plain text and is never bolded or underlined. A continuation line may contain a cloze, and that cloze follows every other cloze rule.
Enzyme and Process Markup (APPLIES TO BULLET TEXT AND THE EXTRA FIELD):
 - Wrap every ENZYME name in angle brackets and every PROCESS or PATHWAY name in curly braces, so the reader can tell at a glance which noun is a catalyst and which is a route: ⟨PFK-1⟩ and {Gluconeogenesis}.
 - USE THE UNICODE ANGLE BRACKETS ⟨ and ⟩ (U+27E8 and U+27E9), NEVER THE ASCII < AND >. An Anki field is HTML, so a literal <PFK-1> is parsed as a custom element tag and the enzyme name renders as NOTHING at all, swallowing the text that follows it into that element. The HTML entity escape is barred too, because entities end in a semicolon and a semicolon inside a field destroys the CSV import. The Unicode characters are plain text: they look like angle brackets, break no parser, and need no escaping. This is not optional.
 - Curly braces are written as the ordinary ASCII { and }. Anki treats only {{c1:: as cloze syntax, so a single brace passes through as literal text and is safe.
 - The markup stays ON the term wherever the term appears, including inside a cloze and inside the underline: {{c1::<u>⟨phosphofructokinase-1⟩</u>}} and {{c1::<u>{gluconeogenesis}</u>}}.
 - Do NOT apply the markup inside the bold card title or the bold subtopic headings. Headings stay clean, so <b>Irreversible Enzymes of Glycolysis</b> is written exactly that way.
 - ENZYMES are the named catalysts and catalytic complexes: ⟨hexokinase⟩, ⟨aldolase B⟩, ⟨G6PD⟩, ⟨HMG-CoA reductase⟩, ⟨carbamoyl phosphate synthetase I⟩, ⟨the pyruvate dehydrogenase complex⟩. A deficiency is named after its enzyme, so write deficiency of ⟨aldolase B⟩ with the brackets on the enzyme only.
 - PROCESSES are the named pathways, cycles, shuttles and metabolic states: {glycolysis}, {the TCA cycle} written as the {TCA cycle}, {beta-oxidation}, {ketogenesis}, {the Cori cycle} written as the {Cori cycle}, {reverse cholesterol transport}. Put the braces around the pathway name only, never around a leading article or preposition.
 - Mark up NOTHING ELSE. Substrates, products, hormones, cofactors, vitamins, transporters, diseases, tissues, cell types and clinical findings all stay bare: glucose-6-phosphate, insulin, biotin, NADPH, GLUT4, Heinz bodies.
 - Brace adjacency still applies: a process name that ends a cloze must not put two closing braces side by side. {{c1::<u>{glycolysis}</u>}} is safe because </u> separates them, whereas {{c1::{glycolysis}}} is broken and must never be written.
Chemical Formulae and Equations (MATHJAX, INLINE DELIMITERS ONLY):
 - Every chemical formula, ion, and reaction equation must be written as MathJax so that Anki renders it properly. Never leave a formula as flattened prose. Writing the remaining 15 percent is phosphate ion as HPO4 2 minus and H2PO4 minus is unreadable and is exactly what this rule exists to prevent.
 - Use Anki inline delimiters only: a backslash and an opening parenthesis to start, a backslash and a closing parenthesis to end. Written out: \(\mathrm{HPO_4^{2-}}\) and \(\mathrm{H_2PO_4^-}\).
 - Do NOT use the display delimiters \[ and \]. They would put square brackets inside the card, and square brackets are reserved for the slide key and the [sa] marker. Display math also breaks the flow of a bullet.
 - Wrap chemical species in \mathrm{} so element symbols render upright rather than italic, which is the correct convention for chemistry.
 - Subscripts use an underscore and superscripts use a caret, with braces around anything longer than a single character: H_2, ^{2-}, PO_4^{3-}.
 - Put a whole reaction inside ONE pair of delimiters rather than stitching several together: \(\mathrm{HPO_4^{2-} + H^+ \rightarrow H_2PO_4^-}\).
 - NEVER use the LaTeX spacing command consisting of a backslash and a semicolon. It places a literal semicolon inside the field and destroys the CSV import, because the semicolon is the column delimiter. Use \, or an ordinary space instead. For the same reason never use a backslash followed by a double quote.
 - A cloze wraps the WHOLE expression from the outside, delimiters included, with the underline inside the cloze as usual: {{c1::<u>\(\mathrm{H_2PO_4^-}\)</u>}}. Never open a cloze inside a MathJax expression, because its braces collide with the LaTeX braces and Anki will mis-parse the card.
Two Closing Braces Will Break a Clozed Formula (THE MOST COMMON MATHJAX FAILURE):
 - Anki finds the end of a cloze by scanning forward for the FIRST pair of closing braces. Any two closing braces sitting next to each other inside a cloze therefore end it early and wreck the card.
 - This bites hardest on ions written with a braced charge inside \mathrm, because the charge brace and the \mathrm brace close together. Clozing \(\mathrm{HPO_4^{2-}}\) makes Anki stop at 2- and leave the closing delimiter stranded outside the blank, so the formula never renders.
 - PREFERRED FIX, close \mathrm before the charge so the two groups never touch: write \(\mathrm{HPO_4}^{2-}\) instead of \(\mathrm{HPO_4^{2-}}\). The two render identically, because a charge contains no letters that italics would affect, and the safe form has no adjacent closing braces at all. Use this form for every ion by default, clozed or not. The same trick applies to a braced subscript such as \(\mathrm{B}_{12}\).
 - FALLBACK FIX, when the structure genuinely cannot avoid a nested group, such as a fraction: insert one space before the outer closing brace, as in \(\mathrm{HPO_4^{2-} }\) or \(\frac{a}{b }\). LaTeX ignores the space and the rendering is unchanged, but the brace pair is broken.
 - The check is simple and absolute: no two closing braces may sit next to each other anywhere inside a cloze. Scan every clozed formula, and every clozed process name, for that pair before emitting the card.
 - Ending the expression with the closing delimiter and the underline tag helps, because a clozed formula then finishes with a backslash, a parenthesis and </u> rather than with braces.
 - Never place punctuation inside a cloze. Writing {{c1::buffer,}} hides the comma along with the word and the revealed sentence reads wrongly. Keep commas, full stops and semicolons outside the braces.
Synonyms and Alternative Names Go in Parentheses (NOT IN A RELATIVE CLAUSE):
 - When a term has a second name, an abbreviation, or an eponym, put it in ROUND BRACKETS immediately after the term instead of spelling out a clause. Write the principal component of the umbilical cord (Wharton's jelly), not the principal component of the umbilical cord, which is also called Wharton's jelly.
 - This replaces every phrasing of the form which is also called, also known as, otherwise called, that is, or in other words. Those clauses pad the bullet and push the real fact further from the eye.
   Tight junctions (zonula occludens) prevent passage between cells.
   Transitional epithelium (urothelium) bears surface plaques.
   Heat production (nonshivering thermogenesis) is its main function.
   Nidogen (entactin) is a basal lamina component.
 - Round brackets are ordinary text and are safe everywhere, unlike square brackets which are reserved for the slide key and the [sa] marker.
 - Keep the parenthesis OUTSIDE the cloze when it merely renames the answer, so the blank tests one name and the bracket reveals the other only when shown. Put it INSIDE when the pair is meant to be recalled together.
 - Do not use this for a definition or a description. A parenthesis carries an alternative NAME, not an explanation, so write ground substance (non-formed connective tissue) but never ground substance (the material that fills the space between fibers).
 - Plain numbers, percentages, doses, and units stay ordinary text. MathJax is for formulae and equations, not for every digit on the slide.
Arrows for Consequence and Reaction:
 - When one thing leads to, produces, converts into, or results in another, write the arrow as two hyphens followed by a greater-than sign: -->. Write raised blood ammonia --> encephalopathy rather than spelling out leads to.
 - This applies to causal chains in prose and to any sequence of steps. Several arrows may appear in one bullet: glutamine --> glutamate --> alpha-ketoglutarate.
 - Inside a MathJax expression use \rightarrow instead, because the two-hyphen form would render there as two minus signs. So prose carries --> and equations carry \rightarrow.
Selective Clozing (NOT EVERY FACT BECOMES A BLANK):
 - Do NOT cloze every fact on the slide. Within each subtopic pick only the two or three highest-yield targets and leave every other fact as plain unclozed prose that supports them. Unclozed prose carries no braces and no underline, though it still carries the enzyme and process markup.
 - Numbers, quantities, doses, and measurements are never placed inside cloze braces and never underlined, even when they are high-yield. Write them as plain text so they are always visible.
 - A subtopic reading that ubiquitin is the marker, that it is 76 amino acids long, and that it targets cytosolic and nuclear proteins should cloze ubiquitin and leave 76 and cytosolic and nuclear as plain text. The blanks carry the idea being tested and the plain text carries the context that makes it answerable.
 - Never use c0 or any cloze number below c1. Anki generates no card for them, so a fact worth marking is either a real numbered cloze or plain text, never a fake one.
Cloze Underlining (EVERY CLOZED ANSWER CARRIES AN UNDERLINE):
 - The answer text of every single cloze must be wrapped in underline tags placed INSIDE the braces, written exactly as {{c1::<u>answer text</u>}}. No cloze is ever left without them.
 - The tags go inside the braces, never outside. Inside, the blank stays clean while that group is being tested, and the answer appears underlined on every card where the group is NOT the one being tested. That underline is the whole point: it marks the words as a former blank so they stand out from ordinary prose instead of reading as plain text.
 - Underline the answer text only. The tags open immediately after the second colon and close immediately before the closing braces, with nothing else between them and the braces. The enzyme or process markup sits inside the underline, as part of the answer text.
Cloze Grouping (GROUP THE TARGETS, DO NOT NUMBER THEM ONE BY ONE):
 - A cloze number identifies a GROUP of facts hidden and revealed together, not a single blank. Anki generates exactly one card per distinct cloze number, so the number of groups on a note is the number of cards it produces.
 - Within one subtopic, if there are only two or three cloze targets, put them ALL in the same group so they share one number, every target under that heading written as {{c1::...}}. They are meant to be recalled together as one idea.
 - If one subtopic holds four or more cloze targets, split them into several groups of roughly two to three targets each, {{c1::...}} across the first pair or triple and {{c2::...}} across the next. Never leave a subtopic as one group of five or more, and never give every target its own number.
 - Aim for two to three targets per group throughout. A group of one is acceptable only when the subtopic genuinely contains a single fact. If a subtopic keeps producing lone groups, that is a sign it should be its own subtopic heading with a single fact under it, which is fine and preferable to padding.
 - Split by meaning, never by position. Keep a contrasting pair, a matched enzyme and product, or a linked cause and effect inside the SAME group, and open a new group where the idea changes. When the count divides unevenly prefer the even split, so five targets become three plus two rather than four plus one.
 - Numbering runs consecutively across the whole card, starting at c1 and never restarting. A later subtopic continues from where the previous one stopped, so a card whose first subtopic used c1 begins its second subtopic at c2. Never reuse a number in a different subtopic and never skip a number.
Repeated Cloze Targets and the [sa] Marker:
 - If a term that has already been clozed appears again later in the SAME subtopic — in a later sentence, in a later bullet, on an asterisk continuation line, or in the bullets beneath a heading you clozed — cloze it again and write the marker [sa] immediately after the closing braces, outside the cloze, exactly as {{c1::<u>Ubiquitin</u>}}[sa].
 - THIS IS EXPECTED BEHAVIOUR, NOT AN EXCEPTION. Repeats are common in lecture prose, and every one of them that genuinely carries the idea forward gets the marker. A finished deck containing NO [sa] anywhere has almost certainly left its repeats sitting as plain text, which is a defect: the second mention is usually the one that shows the reader where the answer actually does its work, so leaving it visible hands the answer over.
 - WORKED EXAMPLE. This bullet clozes the term once and then prints it again in plain text three words later, which gives the blank away:
   - Purkinje fibers sit in the {{c2::<u>subendocardial</u>}} layer, the area between endocardium and myocardium, among subendocardial connective tissue and adipose tissue.
 - Deleting the second occurrence is NOT the fix, because the sentence then loses the word that names the tissue:
   - Purkinje fibers sit in the {{c2::<u>subendocardial</u>}} layer, the area between endocardium and myocardium, among connective tissue and adipose tissue.
 - The fix is to cloze the repeat under the same number and mark it, so both mentions blank and reveal together:
   - Purkinje fibers sit in the {{c2::<u>subendocardial</u>}} layer, the area between endocardium and myocardium, among {{c2::<u>subendocardial</u>}}[sa] connective tissue and adipose tissue.
 - Before emitting any card, scan each subtopic for a clozed term that reappears as plain text within it. Every such occurrence is either marked with [sa] or rewritten away — it is never left standing.
 - The repeat takes the SAME cloze number as its original, so both mentions hide and reveal together on a single card. Never give a repeat a number of its own. If the natural repeat would need a different number, do not use the marker at all — rewrite the sentence instead.
 - The marker sits OUTSIDE the braces so that it stays visible while the answer is blanked, telling the reader at a glance that this blank is an answer they have already met rather than a new fact. Write it lowercase and in plain text, never bolded and never underlined.
 - A repeat does NOT count as a new target when sizing a group. Two distinct targets plus one repeat of one of them is still a group of two.
 - Use judgement about which repeats earn the marker. Mark a repeat when the second mention carries the idea forward or would otherwise leak the answer; do not chain one term mechanically through every bullet of a long subtopic, and never mark a repeat whose original sits in a DIFFERENT subtopic, since the two would then be a single group spanning two ideas.
Front End-Cap Break: You must end the front card text with an HTML line break (<br>) placed immediately before the closing double quote of the field.
Output Ordering (THE FILE MUST ARRIVE PRE-SORTED):
 - Emit the card lines in ascending numeric slide order so the file is already sorted the moment it is imported, and so it reads in lecture order if opened in a text editor.
 - Sort by the numeric value of the number written first in the key. Every key beginning with the same number forms one contiguous group, and inside that group a joined key comes before the plain one. Slides 12 to 14 are emitted as [012], [013-012], [013], [014]. Emit them in that order so the text file matches what the Anki browser will display.
 - When several cards share one slide number, they MUST carry sequence letters, and they are emitted in letter order, which is the order their content appears on that slide, top to bottom and left to right. Keep this order stable so the same input always produces the same output.
 - Never merge, renumber, or reorder cards to make the sequence look tidier. Gaps in slide numbers are expected and must be preserved exactly.
Appending New Cards Later (NO REBUILD REQUIRED): A new card written with the correct zero-padded key needs no adjustment to any existing card and no re-sorting of the deck. It may simply be appended to the end of the file or pasted anywhere, because Anki sorts on the Sort Field rather than on file order, and the padded key places it next to the cards it belongs with. Sort the Anki browser by Sort Field once, and every future addition lands in the right place on its own.
Coverage and the Handling of Content-Light Slides (PREFER MERGING OVER SKIPPING):
 - Skip only pages with NO examinable content at all, typically the title page, section dividers, decorative or devotional interludes, the references page and the closing slide. Name the skipped pages in your response text so the user can see nothing was lost by accident.
 - A page that carries examinable content but too little to justify a card of its own is CONTENT-LIGHT, not empty, and must NOT be skipped. Typical cases are an unlabelled or lightly labelled image plate, a diagram that illustrates the slide before it, and a slide holding a single line of text.
 - Fold each content-light slide into the adjacent card it belongs with, and widen that card key into a range so the picture still reaches the reader: a card built on slide 26 that absorbs the diagram on slide 27 becomes [027-026], and a card on slide 5 that absorbs the image plate on slide 6 becomes [006-005].
 - Merge only into a slide the content actually belongs with, which is nearly always the immediately preceding or following slide. Never merge across a topic change just to avoid a skip, and never merge two slides that each deserve a full card.
 - A ranged card gets ONE img tag per slide it covers, so the merged picture is what the extra slide contributes.
 - Where a run of consecutive content-light slides sits together, one card may absorb the whole run, as [012-010] for slides 10 through 12.

Pre-Output Self-Check (RUN BEFORE EMITTING — DO NOT SKIP)
Verify every line and fix any line that fails before printing the code block. Run the mechanical checks with a script rather than by eye, because a single stray quote silently corrupts an import:
 - Every line begins with a double quote, then an opening square bracket, then exactly three digits, then optionally a - or + joiner with three more digits, then optionally one lowercase sequence letter, then a closing bracket.
 - Any slide that produced more than one card has a sequence letter on EVERY one of those cards, running a, b, c in slide reading order, with no bare unlettered key left among them.
 - Every line carries the lecture name after the key, spelled identically on every card, ending in a colon, followed by <br>. Compare the first line of all cards against each other and fix any that differ.
 - Every line contains exactly two semicolons and exactly six double quotes.
 - No semicolon and no double quote appears anywhere inside a field, no HTML entity appears anywhere in the file, and every HTML attribute uses single quotes.
 - The three columns are in the order Text, Extra, Tags, with Tags last, and every Extra field ends with </font>.
 - Every img filename matches the slide number in the key on that same line, and every referenced file exists in the render folder. Check this by listing the folder, not from memory.
 - Every slide number traces back to a number you actually read on that rendered slide or to that slide PDF page number. If you cannot point to where a number came from, the card does not ship until you re-check the source.
 - Bold tags wrap the title and the subtopic headings only, never a bullet, and no enzyme or process markup appears inside a bold heading. A cloze is the only thing that may sit inside a heading, and where one does the <b> tags still wrap the whole heading from the outside.
 - No cloze anywhere has its answer printed in the bold heading above it or in the bold card title, and any term that was moved up into a heading has been removed from the bullets beneath it or re-clozed there with [sa].
 - No bullet chains a list of separate facts through commas. Every enumerated item, every step of a sequence, and every level of a nested hierarchy sits on its own dash bullet separated by <br><br>.
 - Every contrast or elaboration introduced by whereas, while, unlike or in contrast sits on its own continuation line, opened by an asterisk after a SINGLE <br>, with no comma left joining it to the bullet above. No continuation line carries a dash.
 - Every subtopic has been scanned for a clozed term that reappears in plain text within it, and every such repeat is either marked with [sa] under the same cloze number or rewritten away.
 - Every enzyme name is wrapped in ⟨ and ⟩ and every process or pathway name is wrapped in { and }, and no ASCII < or > surrounds an enzyme anywhere in the file.
 - Cloze numbers run c1, c2, c3 with no gaps and no restarts, and each number covers about two to three targets rather than one.
 - Every cloze answer is wrapped in <u> and </u> INSIDE its braces, with no cloze left bare.
 - Every chemical formula and equation sits inside inline MathJax delimiters with \mathrm for the species, and no display-math brackets and no backslash-semicolon spacing command appears anywhere in the file.
 - Every leads-to relationship in prose is written --> and every arrow inside a MathJax expression is written \rightarrow.
 - No cloze anywhere contains two closing braces sitting next to each other, and every ion is written with its charge outside the \mathrm group.
 - Every range key joined by - has its HIGHER number written first, for example [013-012] and never [012-013].
 - Every content-light slide has been merged into a neighbouring card via a range key rather than skipped, and the only pages absent from the deck are genuinely contentless ones you can name.
 - No slide carries both a lettered plain key and a lettered range key in the wrong order, that is no range card of a lettered slide sorts ahead of a plain card of the same slide.
 - No bullet contains the phrases which is also called, also known as, or that is, used to introduce a second name, since every alternative name sits in round brackets instead.
 - Every [sa] marker sits outside its braces, carries the same number as its original, and has that original earlier in the SAME subtopic. No other square brackets appear outside the slide key.
 - Each Text field ends with <br> immediately before its closing quote.
 - The lines are in ascending numeric slide order.
State the result of this check in one sentence in your response text, along with which numbering source you used, before the code block.

2. EXTRA (Back Field)
Color Enclosure: The ENTIRE text string inside the Extra column must be wrapped completely inside an HTML font tag specifying single quotes for color: <font color='#55aaff'>Supplementary info — sourcecitation</font>
Component Layout: Separate the non-clozed supplemental medical context from the mandatory source citation using an em dash ( — ).
Source Citations Syntax: Format book chapters as booknamech##, journals as journalnameYYYY, and lectures as authornameYYYY. Never use cloze deletion syntax here. Never invent a chapter number you did not read — if the deck cites a book without a chapter, use the book name plus its year instead.
Enzyme and Process Markup: applies here exactly as it does in the Text field.
Slide Image (LAST THING INSIDE THE FONT TAG): end the Extra field with <br><br> then one img tag per slide number appearing in the key, written with single quotes, placed INSIDE the closing font tag: <font color='#55aaff'>context — citation<br><br><img src='ccom-009.jpg'></font>
 - The double break is deliberate: it puts a blank line between the supplementary text and the picture, matching the <br><br> spacing used everywhere else, so the image never crowds the citation.
 - Inside the font tag, not after it, so the rule that the entire Extra string sits within one font wrapper still holds. A font colour has no effect on an image, so nothing is harmed.
 - Filename: prefix from STAGE 1.5, then the slide number zero padded to three digits, then .jpg. The number must match the key on the SAME line.
 - Joined Keys: a range or non-adjacent key gets one img tag per slide it covers, in ASCENDING numeric order, separated by <br>. So [013-012] ends with <img src='xx-012.jpg'><br><img src='xx-013.jpg'>.
 - PRUNING THE IMAGE LIST ON A WIDE RANGE: when a key spans 3 or more slides, you MAY leave out the img of any slide that is pure text whose content you have already written out on the front of the card. Its picture would only repeat the bullets the reader is already looking at. Keep every slide that shows something the front cannot carry, which is any photomicrograph, diagram, table or labelled figure.
 - Never prune below one image, never prune on a key covering only 2 slides, and keep the surviving images in ascending order. Say in your response text which slides you pruned and why.
 - Existence: every filename you write must correspond to a file you actually rendered in STAGE 1. Verify this programmatically before emitting, never by memory.
3. TAGS (Tag Field)
Layout: Exactly one unified lowercase organizational tag per card line. Combine the component shorthand (e.g., bsx, dx, tx, dtx, pgx) with a short disease/topic acronym (e.g., bsxtranscription, txmi).

MODEL EXAMPLE LINES WITH SPECIFIC HEADINGS
"[005] Gene expression:<br><b>Nucleosome Structural Organization</b><br><br><b>Macromolecular Architecture</b><br><br>- Core DNA Length: Each eukaryotic nucleosome unit contains roughly 200 nucleotide pairs of DNA.<br><br>- Core Particle Components: High salt concentrations separate the core particle into a 147-nucleotide-pair double helix and a central {{c1::<u>histone octamer</u>}}.<br>";"<font color='#55aaff'>Individual nucleosome core particles are isolated when a linker-cleaving enzyme called ⟨nuclease⟩ digests linker DNA. — chuaypen2026<br><br><img src='ge-005.jpg'></font>";"bsxchromosome"

"[007] Gene expression:<br><b>The Two Functional Formats of Chromatin</b><br><br><b>{{c1::<u>Euchromatin</u>}}</b><br><br>- Loose chromatin that remains fully open and active for {transcription}.<br>* whereas the condensed format is inaccessible to transcription factors.<br><br><b>{{c1::<u>Heterochromatin</u>}}</b><br><br>- Densely packed, condensed chromatin that is transcriptionally inactive.<br><br>- It is the format that carries the inactive X chromosome in the female nucleus.<br>";"<font color='#55aaff'>Euchromatin is driven by ⟨histone acetyltransferases⟩, leading to hyperacetylated histone tails. — chuaypen2026<br><br><img src='ge-007.jpg'></font>";"bsxchromosome"

"[012] Gene expression:<br><b>RNA Polymerase Subunit Composition</b><br><br><b>Core Enzyme Subunits</b><br><br>- The bacterial core enzyme carries two {{c1::<u>alpha</u>}} subunits.<br><br>- It carries one {{c1::<u>beta</u>}} subunit.<br><br>- It carries one {{c1::<u>beta prime</u>}} subunit.<br><br><b>Holoenzyme Assembly and Promoter Recognition</b><br><br>- Adding the {{c2::<u>sigma factor</u>}} to the core enzyme converts it into the {{c2::<u>holoenzyme</u>}}.<br>* whereas the core enzyme on its own can elongate but cannot start a chain.<br><br>- The {{c2::<u>sigma factor</u>}}[sa] lets ⟨RNA polymerase⟩ recognise the {{c3::<u>promoter</u>}} region and start {transcription} at the correct {{c3::<u>start site</u>}}.<br>";"<font color='#55aaff'>Sigma factor is released shortly after initiation, leaving the core enzyme to carry out elongation on its own. — essentialcellbiologych06<br><br><img src='ge-012.jpg'></font>";"bsxtranscription"

"[010] Amino acid metabolism:<br><b>Metabolic Precursors of the Non-Essential Amino Acids</b><br><br><b>Precursors Drawn From Glycolysis</b><br><br>- Alanine is formed by {transamination} of {{c1::<u>pyruvate</u>}}.<br><br>- Serine is derived from the glycolytic intermediate {{c1::<u>3-phosphoglycerate</u>}}.<br><br><b>Precursors Drawn From the TCA Cycle</b><br><br>- Aspartate is formed by {transamination} of {{c2::<u>oxaloacetate</u>}}.<br><br>- Glutamate is formed by {transamination} of {{c2::<u>alpha-ketoglutarate</u>}}.<br><br>- Glutamine is then made from {{c3::<u>glutamate</u>}} by the enzyme {{c3::<u>⟨glutamine synthetase⟩</u>}}.<br>";"<font color='#55aaff'>Only the carbon skeleton needs a dedicated precursor because the amino group is supplied by {transamination} from the shared glutamate pool. — chuaypen2025<br><br><img src='aam-010.jpg'></font>";"bsxaasynthesis"

"[042] Mineral metabolism:<br><b>Phosphate as the Major Intracellular Buffer</b><br><br><b>Distribution of Buffer Phosphate</b><br><br>- The remaining 15 percent of body phosphate acts as buffer, present as \(\mathrm{HPO_4}^{2-}\) and \(\mathrm{H_2PO_4^-}\), with 14 percent intracellular and 1 percent extracellular.<br><br><b>How the Pair Resists Acid</b><br><br>- Added acid is taken up by the reaction {{c1::<u>\(\mathrm{HPO_4}^{2-} + \mathrm{H^+} \rightarrow \mathrm{H_2PO_4^-}\)</u>}}, so the pair buffers a fall in pH.<br><br>- Because most of it sits inside cells, a large acid load --> phosphate buffering --> a rise in urinary {{c1::<u>titratable acid</u>}}.<br>";"<font color='#55aaff'>Bicarbonate handles most extracellular buffering, which is why phosphate is described as the major intracellular buffer rather than a plasma one. — chuaypen2025<br><br><img src='mine-042.jpg'></font>";"bsxacidbase"


================================================================
STAGE 3 — INSTALL THE IMAGES INTO ANKI
================================================================

A card referencing an image that is not in collection.media shows a broken image forever. Anki does NOT copy media during a text import — it resolves every <img src> against collection.media at review time. The render folder from STAGE 1 is staging only, and the deck is not finished until the files are installed.

3.1 FIND THE RIGHT PROFILE
 - macOS profiles live in ~/Library/Application Support/Anki2/<ProfileName>/collection.media
   Windows: %APPDATA%\Anki2\<ProfileName>\collection.media
   Linux: ~/.local/share/Anki2/<ProfileName>/collection.media
 - Ignore addons21 and logs, which are not profiles.
 - If more than one profile exists, pick the one whose collection.anki2 was modified most recently, and TELL the user which one you chose and why so they can correct you. Never guess silently.

3.2 COPY, DO NOT MOVE OR OVERWRITE
 - Before copying, check whether any target filename already exists. If one does, STOP and ask — an existing file means either a re-run or a prefix collision with another lecture, and overwriting could break a different deck.
 - Copy the files in. Never delete anything from collection.media, and never move the staging folder into place wholesale.
 - After copying, verify programmatically that EVERY distinct filename referenced by the deck now resolves inside collection.media. Report the count. Note that this count is normally lower than the page count, because skipped pages produce no cards.

3.3 TELL THE USER WHAT REMAINS
 State plainly, because none of it is guessable from the file:
 - The file imports into a stock Cloze note type with no field changes: map the columns Text, Extra, Tags, and keep Allow HTML in fields checked.
 - The image rides inside Extra, so it appears wherever the back template already prints that field. On the stock Cloze note type that field is called Back Extra. If the user sees the card but no picture, the cause is almost always that the back template does not print the Extra field at all, or that the files never reached collection.media.
 - If Anki was running while files were copied, run Tools, Check Media once so Anki reconciles its media database.
 - Check Media will list the skipped pages under Unused files. They should NOT be deleted if the user may card those pages later.
 - Only now is the staging folder safe to delete.

Read the attached material thoroughly and execute this structure systematically.
