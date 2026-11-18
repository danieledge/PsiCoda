# A setback: running out of memory

*18 November 2026.*

## What happened

Last night I put the newest Coda on my own netBook and used it properly for an evening. A lot worked well. But it got slower as the evening went on, and it ran out of memory far sooner than it should. A few times it restarted on its own. In the emulator none of that had shown up.

So today is mostly about memory. It's a setback, but an interesting one, and I think it's one I can get past.

## Why memory is tight on the netBook

The netBook has 32 MB of memory and no flash ROM. At every cold start the bootloader copies the whole system off the card, from `OS.IMG`, into that memory, and it stays there. That's what makes it so easy to change, and it's also the cost: **every byte of the system is a byte of memory you can't use**.

Two more things make it worse:

- **It goes in whole megabytes.** The system sets aside memory for the image in whole MB, rounded up. Coda's image was 14 MB with 82 KB to spare. Add 100 KB more and it's a 15 MB image, and you lose a whole megabyte of memory for that 100 KB. The other way round, getting a little under a line gives you a whole MB back.
- **Programs on the card load into memory to run.** Programs in the system run straight from where they sit. Programs on the card are copied into memory when they start: about 2 MB for PsiWeb, 1.4 MB for PsiPDF, 440 KB for PsiMail. That's on top of what they use for their work.

With the 14 MB image, the System screen showed about 15.7 MB free before I'd opened anything.

## Why the emulator didn't warn me

The emulator I test with had a fault of its own. The netBook has places in its memory map where there is no memory, and the emulator answered for those places with a copy of real memory. So the system thought it had 64 MB, and when it used the part that wasn't there, it overwrote its own records. A few crashes I'd been chasing for days were this, not Coda. It also made smaller images look broken when they weren't: taps on the icons beside the screen were ignored.

I've fixed my copy of the emulator so it sees what a real 32 MB netBook sees. Every memory number in this entry comes from that.

<details>
<summary>Technical detail: what was wrong</summary>

The netBook's memory is in banks, and two of the bank addresses have nothing fitted. The emulator mirrored those addresses onto the first bank instead of leaving them empty. At start-up the kernel tests each bank, finds memory in all of them, and builds its page tables for 64 MB. Then a page it hands out in a "mirrored" bank is really a page it already uses for something else. The fix leaves the empty banks empty, and the kernel's count drops to the real figure.
</details>

## What I'm trying

**A smaller system.** I measured which parts of the image are actually read in normal use. In five minutes of opening and using programs, about 7 MB of a 13 MB image was never touched. That's the space to win back:

| System image | Free at the System screen |
|---|---|
| 14 MB (last night's) | 15.7 MB |
| 13 MB (on my card now) | 16.7 MB |
| 10 MB (a test build) | 19.7 MB |

The 13 MB one only leaves out four extra wallpapers. To get to 10 MB, the things you don't use all the time move from the system to the card, as ordinary files that Coda picks up from there. That means:

- Smooth text's styles and the extra typefaces;
- the Office file converters;
- Help;
- Coda's start-up sound;
- both versions of Sketch, Calc and Record.

On the card they use no memory until you open them, and they give it back when you close them. The aim after that is about 9 MB, with the spelling dictionaries and a few old libraries on the card too.

There's a catch: a netBook without its card loses those programs. So PsiBoot will offer two Coda images: the small one, for the most free memory, and a full one that keeps everything in the system and works without the card. Psion's own image stays there as well.

**Programs that hold less.** Some of Coda's own programs set aside big buffers the moment they start, whether they need them or not. Sync holds about 450 KB, PsiMail about 370 KB and PsiWeb about 240 KB. They'll take those buffers when they connect or sync, and give them back after. PsiWeb carries 640 KB of fonts inside the program and PsiPDF about 300 KB. Those can be files on the card, read only when they're needed.

**Telling you when memory runs low.** Running out quietly is the worst way to find out. The plan:

- under 3 MB free, Coda tidies up by itself;
- under 1.5 MB, the free memory line on the toolbar turns over and says "Memory low - tap free memory for choices";
- under 750 KB, or whenever something says "Not enough memory", a Free memory dialog lists what you can do, biggest first: close a program and get so much back, put PsiWeb to sleep, clear stored pages, or move files to the Memory disk.

## What I ruled out

The obvious idea is to use the card as swap, as a PC uses its disk. I went through it properly. EPOC Release 5 never pages memory out, and the system and its servers write straight into each program's memory whenever they like, so a page that has been moved to the card can't be brought back safely. Loading the system from the card a piece at a time as it's needed is possible in principle. But it means rewriting the card driver for the moment the system is stopped, and the netBook would stop dead if the card came out. That's weeks of risky work for about what moving files to the card gives anyway.

What does work is gentler: programs that put themselves to sleep in the background and give memory back. PsiWeb and PsiPDF already do.

## Next

The 13 MB image is on my card to try now. The small image with the choice in PsiBoot comes next, then the programs' savings and the low memory warning. I'll put the numbers from the real netBook here once I have them.
