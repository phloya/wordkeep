# How the deck is built

[Русский](./how-it-works.md)

The deck is a plain Anki package (`.apkg`): one note type, one card template and media files with pictures. Everything below can be opened in Anki desktop via **Tools → Manage Note Types**.

## Note type

"Wordkeep (Oxford 3000)", one card per note (template "EN → RU").

| Field | Content |
|---|---|
| `Word` | the English word |
| `POS` | part of speech |
| `Level` | CEFR level: A1, A2, B1 or B2 |
| `Translation` | Russian translation of the word |
| `Example` | the story fragment as HTML: document label, text, place picture, word picture; on the last card of a scene, the whole scene |
| `ExampleRU` | Russian translation of the fragment (of the whole scene on its last card) and the episode number |

Version 2.0 adds no fields and renames none.

## Fragment markup

The `Example` field of an ordinary card, for example "again" from episode 1 (indented for reading; in the deck it is all on one line):

```html
<div class="strip strip-log">
  <img class="wpic" src="tw_1791142942402_….svg" alt="">          <!-- word picture -->
  <div class="panel art-panel scene-office">…</div>             <!-- place picture -->
  <div class="panel text-panel doc doc-log">
    <div class="story-label"><span class="lbl" data-label="Office log (Ida), day 1"></span></div>  <!-- document label -->
    <div class="ex ex-story">… Then she reads it <b>again</b>. And again.</div>
  </div>
</div>
```

- `strip-<kind>` / `doc-<kind>` — the document kind: `scene`, `log`, `card`, `box`, `receipt`, `report`, `news`, `radio`, `announce`, `letter`, `chat`, `voice`, `poster`, `sticky`, `hand`, `overheard`. It sets the label icon and the text style.
- `scene-<place>` — one of 15 place pictures (the class name is left over from earlier versions): `office`, `city`, `desk`, `typewriter`, `bus`, `box`, `receipt`, `island`, `harbor`, `radio`, `cafe`, `fridge`, `station`, `street`, `phone`.
- `<b>` — the deck word inside the fragment, always exactly once, with a yellow underline.
- The label text sits in the `data-label` attribute and is shown by `.story-label .lbl::before { content: attr(data-label) }`. Text-to-speech reads the field without tags and attributes, so it starts with the fragment itself, not with "Office log (Ida), day 1".
- `strip-finale` — the last card of each part.
- The empty `<i>` and `<u>` elements inside `art-panel` are left over from earlier designs and are hidden by the CSS.

`ExampleRU` has the same structure without the word picture and the place picture, and ends with `<div class="story-meta">Серия N · i/n</div>` (episode N, card i of n).

## The card that ends a scene

Every 3–6 neighbouring cards of an episode form a scene: 734 scenes in 179 episodes. The story text is the same as before; the fragments are just put together. Only the last card of each scene has new markup. Its `Example` field — for example on the card "company", which ends the second scene of episode 1 (simplified and indented for reading; in the deck it is all on one line):

```html
<div class="strip strip-scene strip-sc">
  <img class="wpic" src="tw_1791142942546_….svg" alt="">
  <div class="panel art-panel scene-office">…</div>
  <div class="panel text-panel">
    <div class="sc" hidden>
      <div class="sc-head" data-t="Серия 1 · сцена 2/4"></div>       <!-- scene header -->
      <div class="sc-prev" data-t="Ранее: …"></div>                  <!-- "Previously", if any -->
      <div class="sc-f doc doc-scene">
        <div class="story-label"><span class="lbl" data-label="Lost Property, the office rule"></span></div>
        <div class="ex ex-story">… <b>review</b> …</div>          <!-- fragment of the scene's 1st card -->
      </div>
      <div class="sc-f doc doc-log">…label and fragment of the 2nd card…</div>
    </div>
    <div class="sc-f sc-own doc doc-scene">…label and fragment of this card…</div>
    <details class="sc-who" hidden><summary data-t="Кто здесь"></summary>
      <div class="who" data-n="Бруно" data-t="начальник Иды в бюро находок"></div>
    </details>
  </div>
</div>
```

- `strip-sc` marks the last card of a scene. `strip-<kind>` and the `doc-<kind>` of `sc-own` are this card's own document kind. The `text-panel` itself has no `doc` classes: each fragment carries its own kind.
- `sc-f` is one fragment of the scene, with the same label and `ex-story` as on an ordinary card. Fragments follow story order, each exactly once; a scene has 3 to 6 of them. A fragment whose label repeats the previous one also gets the class `sc-same` (`class="sc-f doc doc-scene sc-same"`), and the repeated label is not drawn.
- `sc-own` is this card's own fragment. It is also what «контекст» shows.
- `sc-prev` is «Ранее» (Previously): one line about an earlier scene of the same storyline (or about the previous scene if there is none). The very first scene has no such line.
- `sc-who` is «Кто здесь» (Who's here): up to 5 characters who appear in the scene's text, each with a short note. Ida is never listed. If nobody matches, the block is left out.

The `ExampleRU` field of such a card works the same way: a `<div class="sc" hidden>` with the Russian fragments of the earlier cards, then `sc-own`, and at the end `<div class="story-meta">Серия N · i/n</div>`. It has no pictures, no scene header, no «Ранее» and no «Кто здесь».

### Why the Russian strings live in attributes

Text-to-speech reads the whole `Example` field with an English voice, after dropping tags and attributes. So `Example` contains no Russian text nodes: «Ранее», «Кто здесь», the scene header, the names and the notes sit in the `data-t` and `data-n` attributes and are drawn by CSS — `::before { content: attr(data-t) }`, just like the document label. Only the English text of the scene is read aloud.

### Why the `hidden` attribute

Everything new inside the field carries the `hidden` attribute. The old template and style (versions 1.x) know nothing about scenes, and under them a scene card looks like an ordinary one: its label and its own fragment. So the deck still works if the template stays old after an update. The new style shows the hidden parts where they belong.

## Templates and style

The files are in [`docs/template/`](./template/):

- [`front.html`](./template/front.html) — the front: part of speech, level, the word and its audio. The `Example` field stays in the markup but the CSS does not show it: the style uses it to recognise the last card of a part.
- [`back.html`](./template/back.html) — the back, see below.
- [`style.css`](./template/style.css) — day and night themes (`.nightMode` and `prefers-color-scheme`), the Literata and PT Mono fonts from Google Fonts, the phone and desktop layout.

The back:

```html
<div class="page back">
  <hr id=answer>
  <div class="meta">…part of speech · level…</div>
  <div class="word">{{Word}}</div>
  <div class="ru">{{Translation}}</div>
  <div class="example ex-main">{{Example}}</div>
  <details class="ctx"><summary>контекст</summary><div class="example ex-ctx">{{Example}}</div></details>
  <details class="ru-hint"><summary>перевод</summary><div class="example-ru">{{ExampleRU}}</div></details>
</div>
<span class="tts">{{tts en_US:Example}}</span>
```

- `ex-main` shows the word picture and the place picture, and on a scene card also the story scene with «Ранее» and «Кто здесь».
- `ex-ctx`, inside «контекст» (context), shows only the card's own fragment, without pictures. On a scene card «контекст» is hidden, because that fragment is already part of the scene.
- The `<hr id=answer>` mark sits at the top: AnkiWeb scrolls to it after "Show Answer", so the word and its translation stay on screen.

What `style.css` does not contain:

- **Place pictures.** The deck's style also has 15 rules `.scene-<place> { background-image: url("data:image/svg+xml,…") }`, with the place-picture SVGs embedded in the CSS. They are left out of the file in `docs` so it stays readable.
- **Word pictures.** These are 3,057 media files (almost every word has one) named `tw_<note id>_<hash>.svg`.

**Text-to-speech.** The front uses `{{tts en_US:Word}}` and the back `{{tts en_US:Example}}`, with the system voice. On an ordinary card the back reads its fragment; on a scene card it reads the whole scene. The document label and the Russian strings are not read aloud (see the attributes above). AnkiWeb has no TTS and prints the tag's content as text, so that text is hidden by `.tts > :not(.replay-button):not(a)`.

## Card order

New cards are positioned in episode order: episode 0 is the first 33 cards, and almost every episode after it has 17. The deck preset "Wordkeep" sets 15 new cards and 200 reviews a day, with new cards shown after reviews. It is applied if you enable importing deck presets.
