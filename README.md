# Coda for the Psion netBook

Finishing what Psion didn't.

The Psion netBook came out in 1999 and its software never quite got finished. Coda is my edition of its operating system, with the rough edges I keep tripping over sorted out one at a time. This is the diary of that work. Each entry has screenshots of the bit that changed, cropped and blown up, with Psion's original next to mine where there is one.

There's no ROM, no ROM files and no source code here. Just the writing and the pictures. That will come once I've got things reasonably stable.

## Key features

- **PsiHub.** Press Ctrl+Menu in any program and a small panel pops up. With nothing typed it flips between programs, most recent first. Start typing and it finds programs, settings, files and commands, and learns what you use. [More](posts/2026-11-14-psihub-and-psistart.md)
- **A Linux terminal.** PsiLinux runs Alpine Linux on the netBook, in colour. Type `term` into PsiHub and you're at a shell. [More](posts/2026-11-14-terminal.md)
- **Proper replacements for the old programs.** PsiMail instead of Psion's Email, PsiWeb instead of the web browser, PsiTerm instead of the old Comms terminal. [More](posts/2026-11-12-the-starting-point.md#what-replaced-what)
- **Six colour themes**, picked in the Control panel with a preview. The toolbar name box, the Documents bar, the wallpaper and both clocks all follow along. [More](posts/2026-11-14-colour-everywhere.md)
- **A clock you can read**, cream rather than yellow by default. [More](posts/2026-11-14-the-clock.md)
- **Battery percentage, free memory and a brightness control** on every toolbar. [More](posts/2026-11-14-the-toolbar.md)
- **SerialBoost**, which runs the serial port at 230,400 bps, twice what Psion allowed. On by default. [More](posts/2026-11-14-small-additions.md#serialboost)
- **PsiView**, which shows each photo in 256 colours picked for that photo. [More](posts/2026-11-14-pictures-and-colour-tests.md)
- **A tidy up of the bits that were rushed in 1999**, from the fizzy dotted desk behind Calc to Time's lurid world map. [More](posts/2026-11-14-the-review.md)

Every change is listed in [STATUS.md](STATUS.md), with a screenshot.

## The netBook

It's a clamshell with a 640 x 480 colour screen, a 190 MHz StrongARM, 32 MB of memory and a CompactFlash slot, running EPOC Release 5, the same system as the Series 5mx.

The handy thing is that it has no flash ROM. At every cold start the bootloader copies the whole operating system off the card into memory, from a file called `OS.IMG`. So a new edition is just a new file on the card, and Psion's own can sit next to it for when I break something.

## Why it felt unfinished

EPOC was written for the Series 5: 640 x 240 and 16 greys. The netBook doubled the height and went to 256 colours, and a lot of the system was carried across as it was. You notice. Some programs still draw a 240 line layout in the middle of a 480 line screen. Icons are greyscale or dithered. The contrast setting shows you 15 greys on a colour panel. Time's world map is bright green on navy. The clock is yellow with a name on it you can't read. None of it stops the machine working, but together it feels half done.

## The name

A coda is the closing passage that finishes a piece of music. Until 14 November 2026 this was called the "Psi edition", and older screenshots still say so. [Why Coda](posts/2026-11-14-the-name-coda.md)

## Entries

| Date | Entry |
|---|---|
| 12 November 2026 | [The starting point: a first image, and what replaced what](posts/2026-11-12-the-starting-point.md) |
| 13 November 2026 | [Themes, and a tap on the program's name](posts/2026-11-13-themes-and-the-name-tap.md) |
| 14 November 2026 | [PsiHub and PsiStart](posts/2026-11-14-psihub-and-psistart.md) |
| 14 November 2026 | [Colour everywhere](posts/2026-11-14-colour-everywhere.md) |
| 14 November 2026 | [The clock](posts/2026-11-14-the-clock.md) |
| 14 November 2026 | [The toolbar](posts/2026-11-14-the-toolbar.md) |
| 14 November 2026 | [Wallpapers](posts/2026-11-14-wallpapers.md) |
| 14 November 2026 | [Icons](posts/2026-11-14-icons.md) |
| 14 November 2026 | [Fonts](posts/2026-11-14-fonts.md) |
| 14 November 2026 | [The review of rushed parts](posts/2026-11-14-the-review.md) |
| 14 November 2026 | [Pictures and colour tests](posts/2026-11-14-pictures-and-colour-tests.md) |
| 14 November 2026 | [Small additions](posts/2026-11-14-small-additions.md) |
| 14 November 2026 | [A Linux terminal](posts/2026-11-14-terminal.md) |
| 14 November 2026 | [The name: Coda](posts/2026-11-14-the-name-coda.md) |
| 14 November 2026 | [What is next](posts/2026-11-14-what-is-next.md) |


## Adding an entry

Note to self, for keeping this going:

1. Take screenshots, crop to the bit that changed and scale it up 2x with nearest neighbour. Put Psion's version next to it if there is one.
2. Save them as palette PNGs in `img/YYYY-MM-DD-slug/`, and keep them small.
3. Write `posts/YYYY-MM-DD-slug.md`: what bugged me and what I did.
4. Add a row to STATUS.md for each change and the post to the table above.
5. Before committing, check the text and every image for names, addresses and paths that shouldn't be public.

British English, short sentences, no dashes.
