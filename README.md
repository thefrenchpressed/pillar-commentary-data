# Pillar Commentary Data

Public-domain Bible commentary text, converted into the same JSON schema used by
[HelloAO's Free Use Bible API](https://bible.helloao.org/docs), for use in
[The Pillar](https://github.com/thefrenchpressed) Bible reading app (and anyone
else who wants it — that's the point of public domain).

## Layout

```
c/{commentary-id}/manifest.json      -- id, name, language, list of covered USFM book codes
c/{commentary-id}/{USFM}/{chapter}.json
```

Each chapter file:

```json
{ "chapter": { "content": [
    { "type": "verse", "number": 16, "content": [ { "text": "For God so loved the world..." } ] },
    ...
] } }
```

Fetch directly via jsDelivr, e.g.:
`https://cdn.jsdelivr.net/gh/thefrenchpressed/pillar-commentary-data@main/c/calvin/JHN/3.json`

## Commentaries

### calvin — John Calvin's Commentaries

Source: [CrossWire SWORD project's `CalvinCommentaries` module](https://www.crosswire.org/sword/modules/ModInfo.jsp?modName=CalvinCommentaries)
(public domain, sourced from [CCEL](https://www.ccel.org/)). Covers 47 books —
see `c/calvin/manifest.json` for the exact list; Calvin didn't write on every
book of the Bible (e.g. no Samuel/Kings/Chronicles, Job, Proverbs, Acts, Revelation).

A handful of verses (Genesis 1:1, Psalm 1:1, 1 Timothy 1:1, 2 Timothy 1:1) are
intentionally omitted: the source module attaches book/chapter front matter
(title pages, translator's prefaces) to those verse slots instead of a
per-verse comment, and it isn't reliably separable from genuine commentary —
shown as missing rather than risk showing the wrong thing.

## License

The underlying commentary text is public domain. This repository's conversion
scripts and structure are released under the MIT License (see `LICENSE`).

### fbmeyer — F. B. Meyer's Through the Bible Day by Day

Source: the transcription served by
[SermonIndex](https://www.sermonindex.net/commentary/fbmeyer/), which already
addresses it by USFM book code and chapter. Meyer died in 1929 and the work was
published 1914–18, so it is public domain in the United States.

Built by `Scripts/import_commentary_fbmeyer.py` in the app repo. 973 of the
Bible's 1,189 chapters, 1,940 daily portions, ~384,000 words, 62 of 66 books.

**It is a devotional commentary and it does not cover everything.** Absent
entirely: 1 Chronicles, Song of Solomon, Lamentations, Nahum. Thin where a
devotional writer would thin out: Ezekiel 9/48, Ecclesiastes 4/12,
2 Chronicles 11/36, Proverbs 16/31, Jeremiah 27/52. The New Testament is
complete — all 27 books, every chapter.

Each portion is stored under its first verse, and its text begins
`Title -- the day's reading` so the app's commentary formatter renders the
title as a bold lemma. The reading is kept rather than dropped because a
portion often spans a chapter boundary (John 3:1-8 is read with John 2:23-25),
which the chapter-scoped block header alone cannot show.

`previousChapterReference` / `nextChapterReference` point at the previous and
next *covered* chapter rather than at n-1 and n+1, so chapter navigation steps
over the 216 gaps instead of into them.

Two source quirks are corrected on import and are worth knowing about if you
re-derive this from the same pages: possessives arrive split (`Lord' s`) and
quotation marks arrive spaced (`" tender grass;"`), and on Psalms pages an
`<hr>` introduces a block from Meyer's separate Psalms exposition that is
repeated after *every* portion in the chapter — each portion's text therefore
ends at its own first `<hr>`, and truncating the whole page there instead would
drop real portions (Psalms 139:14-24 among them).

## Added 2026-11 — later public-domain works

Built by `Scripts/import_commentary_sword.py` in the app repo, which reads each
work from a SWORD module and refuses any module whose license line is not
`Public Domain`. Editions, publication dates, source URLs and each source
site's terms are recorded in the app repo at
`licensing/public-domain-commentaries-2026-11.md`.

| id | Work | Published | Source module | Coverage |
|----|------|-----------|---------------|----------|
| `scofield` | Scofield Reference Notes | 1909, rev. 1917 | CrossWire `Scofield` 2.1 | 948 chapters, 66 books |
| `wesley` | John Wesley's Explanatory Notes | 1755, 1765 | CrossWire `Wesley` 2.0 (beta) | all 1,189 chapters |
| `barnes` | Albert Barnes' Notes on the New Testament | 1832–1853 | CrossWire `Barnes` 1.1 | New Testament, 260 chapters |
| `spurgeon` | C. H. Spurgeon's Treasury of David | 1869–1885 | CrossWire `TDavid` 2.1 | Psalms, 150 chapters |
| `gaebelein` | Arno Gaebelein's Annotated Bible | 1913–1922 | SermonIndex `SIANNBIBLE` | 1,182 chapters, 66 books |
| `maclaren` | Alexander Maclaren's Expositions of Holy Scripture | 1904–1910 | SermonIndex `SIMACLAREN` | 607 chapters, 61 books |
| `vincent` | Vincent's Word Studies in the New Testament | 1887–1900 | SermonIndex `SIVINCENTWS` | New Testament, 260 chapters |

Things worth knowing before relying on any of them:

- **`scofield`, `vincent` and `maclaren` are notes or sermons on single
  verses**, not running commentary. A block belongs to the verse it is stored
  under; a reader that extends a block to the next block's start will show
  Scofield's note on Romans 8:23 for Romans 8:28.
- **`scofield`** — each book introduction is chapter 1's `introduction`. The
  module stores them in the pre-verse slot of whichever verse first has a note.
- **`wesley`** — the module's editorial lines ("Introduction supplied from the
  CCEL edition of Wesley's Notes.") are removed.
- **`barnes`** — "Verse 16." is dropped from the head of each entry; a
  paragraph's leading italic phrase is written `catchword -- comment`; his
  marginal cross-reference lists are left out; an entry that only points back
  to an earlier verse is not written, so the earlier block covers it.
- **`spurgeon`** — Spurgeon's own Exposition only, split at his "Verse N."
  markers, with his preface to the psalm as the `introduction`. The
  "Explanatory Notes and Quaint Sayings" and "Hints to the Village Preacher"
  are not included. Psalm 119:1 has no exposition in the module.
- **`gaebelein`** — a section that crosses a chapter boundary (John
  2:23–3:36) is repeated under verse 1 of the next chapter, when its own text
  names that chapter.
- **Size.** jsDelivr does not serve a GitHub repository larger than 150 MB.
  With these seven the repository is about 115 MB. The Expositor's Bible
  (41 MB) is converted but kept on its own branch for that reason.
