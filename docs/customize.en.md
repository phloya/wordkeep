# Customize it

[Русский](./customize.md)

Style changes are made in Anki desktop: **Tools → Manage Note Types** → "Wordkeep (Oxford 3000)" → **Cards…** → the **Styling** tab. Paste a rule at the very end; it reaches all your devices after a sync.

All recipes below were tested on real cards of the deck: an ordinary card and the last card of a scene, in the day and night themes.

Since version 2.0 the front shows only the word by default, and the story fragment is folded on the back under «контекст» (context). The old "word only on the front" recipe is no longer needed.

## The fragment back on the front

As in versions 1.x: the word and its story fragment on the front.

```css
.front .example { display: block !important; margin-top: 28px; }
```

The front then shows the document label and this card's fragment — no picture, no place picture and no story scene. On the last card of a scene the label is smaller, as inside a scene. The scene itself still waits on the back.

## «контекст» open by default

Styling cannot do this: a folded `<details>` opens only through an attribute in the template. So you have to edit the template itself:

1. **Tools → Manage Note Types** → "Wordkeep (Oxford 3000)" → **Cards…**
2. Open the **Back Template**.
3. Find `<details class="ctx">` and change it to `<details class="ctx" open>`.

You can open «перевод» (translation) the same way: `<details class="ru-hint" open>`. The last card of a scene has no «контекст»: its fragment is already part of the scene.

Keep in mind: once you edit the template, deck updates no longer change it for you. When a new version brings a different template, copy it by hand — see [If the template did not update](#if-the-template-did-not-update).

## No «Ранее» and «Кто здесь»

```css
.sc-prev { display: none !important; }   /* the «Ранее» (Previously) line above a scene */
.sc-who { display: none !important; }    /* the «Кто здесь» (Who's here) list below a scene */
```

You can keep either line on its own. The scene itself stays.

## No pictures and place pictures

```css
.wpic, .back .art-panel { display: none !important; }
.back .word, .back .ru { margin-right: 0 !important; min-height: 0 !important; }
```

The second line removes the space kept for the picture. The story scene, «контекст» and «перевод» stay.

## New cards per day

Deck gear → **Options** → **Daily Limits** → **New cards/day**. The default is 15, with new cards shown after reviews. Any number works: a scene arrives once all its words are done. If a day ends in the middle of a scene, it arrives the next day, after the rest of its words. To read more on a given day, raise the limit for today.

Do not randomize new cards: their order is the order of the story.

## If the template did not update

If you ever edited the card template, updates leave it alone. It is easy to tell: after moving to 2.0 the front still shows the story fragment and the back has no scenes. The deck keeps working as before, just without scenes.

To get the new look, copy the template by hand:

1. **Tools → Manage Note Types** → "Wordkeep (Oxford 3000)" → **Cards…**
2. Replace the whole **Front Template** with [`template/front.html`](./template/front.html).
3. Replace the **Back Template** with [`template/back.html`](./template/back.html).
4. Save and check a card. Re-apply your own edits if you still want them.

<!-- TODO before publishing: decide what users with an edited Styling tab should do. docs/template/style.css omits the 15 embedded place-scene images (see how-it-works), so pasting it would lose the place scenes. Either ship a complete stylesheet for copying or add instructions here. -->

## What cannot be changed

- **Translations are Russian only.**
- **Fonts load from Google Fonts.** Offline, Anki falls back to Georgia, PT Serif and Consolas. The cards still work but look plainer.
