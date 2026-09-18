# HL7 Interface Javascript Builder

A single-file, zero-dependency web app that **generates Mirth Connect transformers from HL7 v2.5.1 messages**.
Drag a field onto another field and it writes the JavaScript for you.

**Live:** https://coffeemilktea.github.io/HL7-Interface-Javascript-Builder/

No build step, no server, no npm. One HTML file — open it locally or host it anywhere.

---

## What it does

Paste an HL7 message, map fields visually, and copy out a working transformer.

1. **Source message** — the message is parsed into a clickable tree annotated with HL7 v2.5.1 field
   names and datatype-aware component names, so `PID-5.2` reads as *Given Name* rather than a number.
   Handles repetitions (`~`), components (`^`), repeated segments, and the MSH separator quirks.
2. **Mappings** — build rules by dragging, clicking, or picking from the recipe library.
3. **Output** — three live views: generated JavaScript, a pasteable channel `<transformer>` XML block,
   and a preview that actually runs your mappings on the sample message and highlights what changed.

## Building a mapping

| Gesture | Result |
|---|---|
| Drag field → field | Creates the mapping immediately |
| Drag field → a card's From/To slot | Retargets an existing mapping |
| Drag field → the drop zone | Starts a new mapping, then click the target |
| Click a field, then click another | Same thing without dragging |
| Drag the ⠿ grip | Reorders mappings |
| Drag a transform pill onto another | Reorders the transform chain |
| Double-click a value in the tree | Edits the sample message in place |
| Hover a mapping (or a field) | Highlights the other side of the link |

Every field in the tree carries a badge showing how many mappings **read** it (taro) and how many
**write** it (thai tea), so you can see at a glance what a message is already wired into.

Sources can be a **field**, **literal text**, a **template** (`${PID-5.2} ${PID-5.1}`), a **channelMap
variable**, the **current timestamp**, or a **new UUID**. Targets can be a field, a channelMap or
connectorMap variable, or "clear field".

## Seeing what a mapping actually does

Each card shows the real values from the loaded sample, live, as a strip under the slots:

```
DOE  →  Doe  →  Doe, J   →  Doe, J        ← final value written
     Title Case  Suffix     ( JANE )      ← struck through: the value being replaced
```

One chip per transform, so a chain that ends up wrong shows you exactly which step broke it. When a
mapping does not run, the strip says why — `disabled`, `condition not met` (with the value the
condition actually read), `not yet applied`, or the error.

The **Preview** tab has a scrubber that applies the first *N* mappings so you can walk the message
forward one step at a time, watching the output change and the pending cards dim. `←` / `→` step it
from the keyboard. Copy respects the step, so you can lift the message as of any point in the chain.

## Transforms

Chain any number of them per mapping; each is applied in order and reflected in both the generated
code and the live preview.

`Uppercase` · `Lowercase` · `Trim` · `Title Case` · `Substring` · `Truncate` · `Replace` ·
`Regex extract` · `Digits only` · `Pad left/right` · `Prefix` · `Suffix` · `Reformat date` ·
`Lookup table` · `Default if empty`

Each mapping can also carry a condition — *only if `OBX-11` equals `F`* — which becomes an `if` block
in the generated code.

## Recipe library

33 ready-made transformers across 7 categories, searchable, each previewing its exact mappings
before you add them:

- **Demographics** — uppercase or title-case names, normalize sex and marital status codes
- **Identifiers** — zero-pad the MRN, site prefixes, assigning authority, fresh UUID control ID
- **Dates** — DOB to `MM/dd/yyyy`, HL7 timestamp to ISO, restamp `MSH-7` to now
- **Orders** — swap placer/filler order numbers, map modality codes, flag abnormal results
- **Routing** — route by facility, patient name into a variable, connectorMap stashing
- **Privacy** — de-identify demographics, mask the MRN, blank the SSN
- **Cleanup** — strip phone formatting, normalize patient class, trim whitespace

## Generated code

Idiomatic Mirth E4X, with helpers emitted only when a rule needs them and lookup tables hoisted
into named constants:

```javascript
// ---- helpers ----
function _v(node) {
  return (node === undefined || node === null) ? '' : node.toString();
}

// ---- lookup tables ----
var LOOKUP_1_1 = {
  'INPATIENT': 'I',
  'OUTPATIENT': 'O'
};

// ---- mappings ----
// [1] PID-5.1 (Uppercase)  ->  PID-5.1
msg['PID']['PID.5']['PID.5.1'] = _v(msg['PID']['PID.5']['PID.5.1']).toUpperCase();

// [2] OBX[2]-8 condition
if (new RegExp('^(H|HH|L|LL|A|AA)$').test(_v(msg['OBX'][2]['OBX.8']))) {
  channelMap.put('abnormalResult', 'Y');
}
```

Paste it into a JavaScript transformer step, or use the **Channel XML** tab to import a whole
`<transformer>` element.

## Keyboard

| Key | Action |
|---|---|
| `/` | Focus the field filter |
| `n` | New mapping |
| `r` | Recipe library |
| `1` `2` `3` | JavaScript / Channel XML / Preview tab |
| `←` `→` | Step through mappings (Preview tab) |
| `⌘Z` / `⇧⌘Z` | Undo / redo |
| `Esc` | Cancel a pick, close a dialog |
| `?` | Shortcut list |

Undo covers everything that changes state — mappings, transforms, conditions, reordering, recipe
adds, imports, and edits to the sample message. Typing is coalesced, so one undo takes back a whole
edit rather than one character.

## Saving and sharing

The `⋯` menu holds **Save** / **Load** (browser `localStorage`) and **Export** / **Import** as a
`.json` file — the export carries the mappings and the sample message together, so a mapping set can
be handed to someone else or checked into a repo. Imported mappings are appended and re-keyed, never
overwriting what is already on screen.

## Path syntax

```
PID-5.1        segment, field, component
PID-3[1].4     second repetition of PID-3, component 4
OBX[2]-8       third OBX segment, field 8
```

Mirth's HL7 XML representation splits down to components, not subcomponents — use a `Replace` or
`Regex extract` transform if you need to reach inside a `&`-delimited value.

## Sample messages

ADT^A01 (admit) · ADT^A08 (update) · ORM^O01 (order) · ORU^R01 (result) · SIU^S12 (scheduling).
Or paste your own — click **✎ Edit** to open the raw message box.

## Theme

Tokyo Night in both modes — Tokyo Night proper for dark, Tokyo Night Day for light, toggled from the
topbar and remembered in `localStorage`. The colours live in one token block at the top of the file;
the layout CSS only ever refers to token names, so retheming means editing that block and nothing
else. Every foreground clears WCAG AA (4.5:1) against all four surfaces in both modes — `--surface2`
is the tightest, so check there first.

Note this page no longer tracks the main site's boba palette, so a site retheme will not reach it.

## Notes and limits

- Assigning to a field absent from the inbound message **appends it at the end of the segment**;
  E4X does not know the schema position. Use an outbound template if strict field ordering matters.
- The Channel XML tab targets Mirth **4.5.0**; edit the two `version` attributes for older servers.
- The preview runs a JavaScript reimplementation of the transforms, so it is a faithful check of your
  mapping logic — but the authority is always Mirth itself. Test in a channel before you rely on it.
- Mappings are applied in order against a message that earlier mappings have already changed, which is
  what the generated code does to `msg`. So a mapping that reads a field an earlier one wrote sees the
  new value, in the preview and in Mirth alike.
- Mappings save to `localStorage` via the **Save** / **Load** buttons; nothing leaves the browser.

## Running locally

Open `index.html` in a browser. That is the whole install.

For a local server:

```bash
python3 -m http.server 8090
# then http://localhost:8090
```
