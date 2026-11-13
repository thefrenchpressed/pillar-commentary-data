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

**A section is stored once, under its first verse.** The SWORD module links one
section to every verse it covers — a whole psalm's exposition to each of its
verses — and the first conversion wrote it out under all of them: 14.6 MB of
the same text again, which took this repository past the 50 MB that jsDelivr
will serve. A reader should find the comment for a verse as *the last block
that starts at or before it*; The Pillar does on both platforms.
`Scripts/dedupe_commentary_sections.py` in the app repo made the change and
checks, verse by verse, that the lookup returns the same text as before.

**Keep this repository under 50 MB.** Over it, jsDelivr answers
"Package size exceeded the configured limit of 50 MB" with a 403 for any file
an edge has not cached. Measure before adding anything:
`git ls-tree -r -l HEAD | awk '{s+=$4} END {print s/1e6 " MB"}'`.

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

### scofield — Scofield Reference Notes (1917)

*Listed in the app as "C. I. Scofield Bible Commentary".* The catalog name is chosen for the app's title
bar, not as the work's title: builds before 1.12 shorten a name only by
stripping an ending such as " Bible Commentary", and the work's own title left
a 30-character label that pushed the book's name off the bar. Name any new
catalog entry the same way. The chapter files still carry the earlier name in
their `commentary` block; the apps do not read it.

C. I. Scofield's notes from the 1917 edition of the Scofield Reference Bible
(Oxford University Press, New York; first edition 1909). Published in the
United States in 1917: public domain. Not the 1967 New Scofield, which is in
copyright. Source: CrossWire's SWORD module `Scofield` 2.1, whose text is the
Wikisource transcription. 948 chapters, all 66 books.

These are **notes on particular words**, not a running commentary: a block
belongs to the verse it is stored under. A reader that extends a block to the
start of the next one will show the note on Romans 8:23 for Romans 8:28. Each
note is written `catchword -- comment`. Each book introduction is chapter 1's
`introduction`; the module keeps them in the pre-verse slot of whichever verse
first has a note. The digital text omits at least one note found in print
(Genesis 1:1, "created").

### gaebelein — Arno Gaebelein's Annotated Bible

*Listed in the app as "Arno Gaebelein Bible Commentary".* The catalog name is chosen for the app's title
bar, not as the work's title: builds before 1.12 shorten a name only by
stripping an ending such as " Bible Commentary", and the work's own title left
a 30-character label that pushed the book's name off the bar. Name any new
catalog entry the same way. The chapter files still carry the earlier name in
their `commentary` block; the apps do not read it.

Arno C. Gaebelein, *The Annotated Bible*, nine volumes, New York: "Our Hope",
1913–1922: public domain. Source: the SWORD module SermonIndex builds of its
own pages (`SIANNBIBLE`). 1,182 chapters, all 66 books.

Gaebelein comments by passage. A passage that crosses a chapter boundary (John
2:23–3:36, Revelation 19:11–20:6) is repeated under verse 1 of the next
chapter when its own text names that chapter — 89 such — so that chapter is
not left empty. SermonIndex's reference linker has rewritten some of his
outline references, occasionally wrongly, and a few headings run into the
sentence after them; the prose is intact.

Both were built by `Scripts/import_commentary_sword.py` in the app repo.
