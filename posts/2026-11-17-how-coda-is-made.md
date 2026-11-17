# How Coda is made

*17 November 2026.*

People have asked how any of this works, given there's no source code for the netBook's operating system. Here is the whole process, from Psion's own image to a card in the machine, and the setup I use to build and test it.

## Starting from Psion's image

The netBook doesn't keep its operating system in a chip. It loads it from the CF card at switch-on: a single file, OS.IMG, about 14 MB. Coda starts from Psion's last English release, v1.05(450), with the community c1.60a patch by Poke^Patisoners (2005), which already fixed a handful of Psion's own bugs: colour icons, the console's colours and so on.

That file is a ROM image: a tree of folders and files, laid out at fixed places in memory. The programs in it run straight from the ROM, where they sit, without being copied into RAM. That's what makes the job hard. Psion built each program to run at one exact address and kept no record of the addresses inside it, so normally nothing in the image can be moved.

<details>
<summary><b>Technical detail: the image format</b></summary>

- OS.IMG is a 256-byte "EPOCARM ROM" wrapper followed by the ROM itself. The wrapper records the ROM's size and its own length; the ROM header that follows gives the address the ROM is built to run at, its size (iRomSize) and a pointer to the root folder list.
- A folder is a length word followed by entries of { size, address, attributes, name length, 8-bit name }, each padded to 4 bytes. The length word counts the entries only, not itself: the kernel walks from just after it, and getting that wrong is an instant data abort at boot. The file server searches entries in order, so they don't need sorting.
- Nothing checks the header's checksum. The community images change content and leave it alone, and so does Coda.
- Programs in the ROM are "execute in place": their code runs from the ROM's own address, and there is no relocation table, which is why nothing stock can move.
- The c1.60a patch fixes are made in place inside Psion's files, so the layout of the image is unchanged from Psion's 450 build.

Tools: **romlib** reads and writes the whole ROM tree, and converts programs to execute-in-place images as it places them; **rom8** lists, extracts and replaces single files, and is the everyday tool for looking inside an image.
</details>


## Taking things out

The first step is making room. Opera and Java came out (nearly 5 MB between them, and PsiWeb and PsiLinux do their jobs better now), along with Psion's old Email program, the plain FTP program, the old Comms terminal, the fax programs, which crashed without Email anyway, and a few viewers Word no longer needs.

Each removal is checked first. The build tool looks at what every remaining program imports and refuses to take out anything still in use, naming the program that needs it. Messaging stays, for example, because Contacts' "Send" uses it even with Email gone.

<details>
<summary><b>Technical detail: checked removals</b></summary>

- Every EPOC program and DLL carries a DLL reference table: the libraries it imports and the ordinals it uses from each. Before anything is removed, the tool compares every staying program's table with the entry points and tables of everything going. If a staying program would lose an import, the build stops and names it.
- What went, roughly: Opera and its engine (2.3 MB), Java with its recognisers (2.6 MB), the Email program (134 KB; the messaging engine underneath stays for SMS and Send), FTP (144 KB), Comms (57 KB), Psion's example files (272 KB, replaced by Coda's own), the fax programs (240 KB), the wireless-card status program, and the .doc and .txt viewers (28 KB) now that Word opens those files itself.
- A path that isn't in a given base is skipped, so the same removal list works on other v1.05 builds.
</details>


## Putting things in

The build tool reads the image's tree, takes things out, adds Coda's programs, pictures and settings, then writes the tree back. It keeps a few strict rules:

- **Psion's files stay exactly where they are.** Nothing Psion built is moved, so nothing breaks.
- **New files go in the gaps,** biggest first, each into the smallest gap it fits. A program has to be in one piece, and PsiWeb alone is 2 MB.
- **The folder lists are written fresh,** pointing at the new files, while the original lists the system still points into are left alone.
- **Nothing may overlap.** If any two files would share a byte, the build stops.

If it all doesn't fit, it drops the least important extras in a set order (spare wallpapers first, then extra typefaces) and tries again, and says what it left out.

<details>
<summary><b>Technical detail: how the image is written</b></summary>

- Files that stay are never moved. Files replaced by something the same size or smaller are written in place and keep their address, so anything pointing at them still works.
- Space freed by removals, and the space between the end of the image and iRomSize, becomes a list of free areas. Neighbouring areas separated only by alignment padding are merged (Opera's and Java's 4.7 MB become one run).
- New files are placed biggest first, best fit, because each ROM program must be one contiguous block.
- Every folder is written afresh into free space and the root list repointed. The original folder blocks are never overwritten, because the kernel keeps pointers into them from the ROM header.
- Before writing, every file's byte range is checked against every other; any overlap refuses the image.
- When there isn't room the build drops extras in a fixed order: spare wallpapers (about 250 KB each), then extra typefaces, then font choices, then the smooth-text styles, and tries again, printing a note for each.

Tools: **mkimage** runs the whole build: removals, resource edits, pictures, the theme, Coda's programs from their installers, the hooks, the write, then the patches. Every step prints a note, so the build log is a full list of what changed. **cardkit** assembles the card folder: the image, the bootloader picture, installers, the card-resident programs and Psion's original image beside them. **branding** makes the 640 x 540 bootloader picture, including the strip the bootloader draws its "Loading" bar from.
</details>


Coda's programs are mostly built to run from the ROM too, so they take no RAM for their code. The bigger ones (PsiLinux, PsiPDF and others) live on the card instead and are installed or run from there.

## Room and memory

The ROM's size decides how much RAM you get: whatever the ROM doesn't use of the memory bank it's loaded into is RAM, in whole megabytes. On a 32 MB netBook a 14 MB ROM leaves about 18 MB. So freeing space inside the ROM gives Coda room for programs; it only gives you more RAM once the ROM's end drops past a megabyte boundary.

There's a way to go further. Psion released the same v1.05 software in several languages, and the same program sits at a different address in each. Comparing two builds word by word shows which words are addresses: a word that differs between them, and points to the same place in each, is an address. With five builds as witnesses that recovers the missing records for most of the image. Rebuilding a program at another build's address with them gives exactly Psion's bytes, every time (607 out of 607). It turns out someone did the same on the Series 5mx about twenty years ago. It works on the emulator as a prototype; it isn't in Coda's images yet.

<details>
<summary><b>Technical detail: ROM size, RAM and moving Psion's code</b></summary>

- The kernel reads iRomSize from the ROM header alone and rounds it up to the next megabyte, because the ROM is mapped in 1 MB sections. The rest of that 16 MB memory bank becomes RAM. The wrapper's size field isn't used for this.
- iRomSize still has to cover every file: with the header saying 12.5 MB and files reaching 13 MB, the System screen lost its icons, because the file server uses the unrounded value as the end of the ROM drive.
- In the emulator a 14 MB ROM against a 16 MB one changed total memory from 50 MB to 48 MB. On a real 32 MB netBook, 14 MB should leave 18 MB of RAM; that figure is worked out, not yet measured.
- **Recovering the addresses.** The v1.05 builds for the UK, US, France, Germany, Spain and the MalayBook are six witnesses. Compare one file across two builds: a word that is identical isn't an address (or points at something that didn't move); a word that differs must point into the same file at the same offset in both builds, or it isn't an address at all. Add what each file's own structure gives anyway (header fields, import table, DLL reference table, export directory) and you have its relocations.
- Results: 222 ROM programs in the 450 build, 331,178 address words found. Recover from some builds, rebuild at the address a held-out build used, and compare: 607 of 607 rebuilds byte-identical to Psion's. The 92 lowest files (the kernel side, the file server, the drivers, Eikon) sit at the same address in all five 45x builds, so there is only one witness for them; they are left where they are, and nothing needs them moved.
- Traps the tool handles: text that happens to look like an address (the four letters "DHCP" in one network driver, for instance; 91 such words in all), the import table's length, which isn't in any header, pointers to a file's very end, and files that are a different version in one build.
- A third-party Series 5mx Pro image in the community collection, 11 MB where Psion's are 16 MB, turns out to have been laid out afresh the same way; predicting it from Psion's own five 5mx builds gives 193 of 194 files byte-identical. So it was done on real machines around 2004.
- **Moving files down.** The prototype packs a base image: it removes Opera, Java, FTP and the wireless status program, then moves everything above 9.5 MB down (186 data files and 23 programs, 13,332 address words rewritten). It refuses to move any program that two builds haven't moved, any whose rebuilds aren't all identical, and any Coda has patched. Then it checks every word of the finished ROM: anything still pointing into a moved file's old place stops it. The count is zero.

Tools: **relocdiff** recovers relocations from the witness builds; **rebase** moves programs and rewrites their addresses; **shrink-check** boots an original and a packed image through the same scripted sessions in the emulator and compares the screens pixel for pixel.
</details>


## Changing Psion's programs without their source

There's no source code for Word, the System screen or the rest, so they're changed in four ways, from gentlest to deepest:

1. **Resources.** Menus, dialogs, texts and lists live in resource files, which can be edited and put back. The System screen's start list, the silkscreen icons and many of Coda's texts are done this way.
2. **Pictures and colours.** Icons, wallpapers and the colour schemes are just files in the image, replaced with new ones.
3. **Small byte changes.** Where a program does one wrong thing, the one instruction that does it is changed. Every change finds its target by its content, not by a fixed address, so it can't land in the wrong place.
4. **Hooks.** To add behaviour, one instruction in Psion's code becomes a jump to a tiny piece of code of ours. It does what the replaced instruction did, calls one of Coda's libraries, then carries on where it left off. That's how the toolbar gets its widgets and clock faces, how text gets smoothed, how themes are applied, and how Ctrl+Menu opens PsiHub. Each hook passes on anything it doesn't handle, so Psion's own behaviour is unchanged.

<details>
<summary><b>Technical detail: resources, patches and hooks</b></summary>

- **Resources** are compiled .RSC files: arrays of records for menus, dialogs, texts and lists. The tools decode them, change or add records, and write them back. The System screen's two UID lists (the silkscreen icons and the start list) and the Eikon colour list behind the themes are edited this way.
- **Byte patches** run after the image is written, because only then are the final addresses of Coda's own files known. Each one locates its target by searching for the surrounding instruction bytes, and refuses to apply if the bytes aren't exactly as expected. Stock code never moves, so a patch found once is found the same way on every build of that base.
- **Hooks**: one instruction in Psion's code becomes a branch to a small stub stored as a file in the ROM. The stub saves the registers it needs, does what the replaced instruction did, calls an exported function in one of Coda's DLLs (also in the ROM), and returns to the instruction after the branch. The stubs are a few dozen to a couple of hundred bytes of ARMv4 assembly; their address words are filled in after the image is written.
- Examples:
  - **the toolbar:** Eikon's control factory, CreateByTypeL. Its "any other type" path now hands Coda's two new control types to Coda's toolbar library, and every other type goes on exactly as before. That's where the widgets and the clock faces come from.
  - **smooth text:** the graphics library's text drawing, at the call that draws each character. Coda's font library draws what it handles; everything else is drawn by EPOC as before.
  - **themes:** where Eikon stores its colour list at start-up. Coda overlays the chosen theme from a settings file.
  - **PsiHub:** Eikon's task list call, so Ctrl+Menu opens PsiHub when it's running, and the stock task list when it isn't.
  - **Open with:** the System screen's open-file function, after the full filename is built.
- **Keys:** Coda only ever takes Ctrl+Fn+letter and Ctrl+Menu globally. New Ctrl keys in Psion's programs are only given letters the program's own hotkey table leaves free, checked when the image is built.

Tools: **keytest** generates a standard block of keys and taps (cut, copy and paste between programs, undo, find, zoom, the Command icons and so on), runs it against Psion's image and Coda's in the emulator, and puts the results side by side. Anything that differs is either by design, and written down, or a bug. **sheetmenus** checks a program's hotkey table for free letters before a new shortcut is added.
</details>


Every key and Command icon is checked afterwards against Psion's own image with the same script, so the shortcuts you know keep working.

## My setup

**The machine.** Everything is built on a Linux box with no special permissions, in a Docker image based on Ubuntu 24.04. It holds only the tools; the SDK sits outside it so builds are quick.

**The SDK.** Psion's EPOC Release 5 C++ SDK, the one Symbian shipped in 1999-2000, in the Linux packaging from the psion_cpp_sdk_linux project. Its compiler is Psion's GCC 3.0 (psion-98r2) targeting ARM, with Psion's binutils 2.9 (psion-98r2). Psion's own Windows tools from the SDK (the resource compiler, the bitmap converter, the installer maker) run under Wine 9.0. Everything is built for the ARMv4 instruction set of the netBook's StrongARM, and also runs on the Series 5mx's ARM710T.

**Other tools.** GCC 13.3 builds the host-side tests and tools. The image tools themselves (reading and writing ROM images, building the image and the card, checking keys) are written in TypeScript and run on Node 22. Pictures, icons and the boot screen are made with Python 3.12 and Pillow. The few tiny hook stubs are written in ARM assembly.

<details>
<summary><b>Technical detail: building a program</b></summary>

- Each program is a project with an .mmp file. The SDK's makmake turns it into a makefile; GCC compiles each source file to ARM objects; dlltool and ld link against the SDK's import libraries; petran converts the result into an EPOC executable with its UIDs.
- Resources are compiled with rcomp, bitmaps with bmconv (four-grey or 256-colour, with separate masks), and the installer is made with makesis from a .pkg file. Coda's programs carry two-digit version numbers, because the EPOC installer only compares major.minor.
- Everything on the ROM drive has to be an execute-in-place image, as all 185 of Psion's are; an ordinary program put there stops the boot at the splash screen. So the image tool does what Symbian's own ROM builder did, at the moment a program is placed: it relocates the code to its ROM address, fills each import slot with the real address of the export it needs (read from that DLL's own place in the same image), writes the DLL reference table after the code, and writes the ROM image header.
- A program with writable data keeps its initial data in the ROM after its code, and it is copied into the process at start, as Psion's own Lexicon and Contacts servers do. A DLL can't have writable data, so a library that needs it is built as a program instead (PsiPDF's engine is one).
- One source tree builds for both the netBook and the Series 5mx: the differences between the machines (screen size, colour) sit in a small layer chosen at build time.
- The netBook's StrongARM SA-1100 has no Thumb and no BX instruction, which modern compilers and libraries use to return from functions. Everything is built without it, the build checks every program for stray BX, and hand-written stubs return with a plain move to the program counter.
- Builds run in a container with Wine, so they need no root and nothing installed on the host beyond Docker.
</details>


**The emulator.** Testing happens in the psionEmulators project (github.com/joehaines/psionEmulators), an emulator for Psion's machines, with its native test harness built with full optimisation. It boots the netBook's real bootloader with Coda's card, about 90 seconds of simulated time to the System screen, and can be driven from a script: taps on the screen, key presses (chords too), screenshots in colour, and a copy of the RAM to read files and logs back out. Every feature here was tested that way, in each colour theme, and each one gets its screenshots there: the pictures on this blog all come from it.

<details>
<summary><b>Technical detail: testing in the emulator</b></summary>

- The harness loads the netBook's v0.11 bootloader and a card image, boots for a set time, then plays a script: taps at screen coordinates, key presses with modifiers, drags, all at given simulated times. It saves screenshots (greyscale, or colour on request), can save the whole RAM, and can log every processor exception.
- One of the emulator's helper hooks for mounting the CF card deadlocks the file server with some layouts of the ROM tree, so it's always switched off for Coda. That's the emulator's fault, not the image's.
- The raw OS.IMG boots to the System screen in about 93 simulated seconds; a full FAT16 card image takes about 180.
- Files, logs and the kernel's own tables are read back out of the saved RAM. That's how Sketch's file and clipboard formats were worked out, and how the kernel's memory blocks were checked during the ROM work.
- Test screens are compared pixel for pixel against saved reference screens, with the clock masked out, so a change that moves anything unexpectedly shows up at once.

Tools: the emulator scripts **run**, **seed** and **mkcard** (already public, in the shared Psion core library) drive the harness, seed test data and build test cards.
</details>


**The ROMs for testing** are Psion's own builds, kept with the emulator: the netBook's v1.05 in its several languages (also the witnesses for the address work above) and the Series 5mx's v1.05.

**What the emulator can't show.** It doesn't use the boot card as a second disk after starting, it doesn't model the netBook's passive colour screen, and it can't do networking, infrared or sound recording. So anything to do with the card, the real screen colours or the network gets its proper test on my own netBook, and the diary says which have.

## From build to netBook

A build produces the new OS.IMG, the bootloader's picture and a folder with everything else the card needs: installers, the programs that live on the card, and Psion's original image kept beside it. Copy that onto the card, switch on, and the netBook loads Coda. To go back, rename Psion's original to OS.IMG and take the battery out and back in.

## The tools

Everything named in the technical sections above was written for Coda: romlib and rom8 for reading and writing ROM images, mkimage and cardkit for building the image and the card, branding for the boot picture, relocdiff, rebase and shrink-check for the ROM-shrinking work, keytest and sheetmenus for keeping Psion's keys intact, plus the emulator scripts. The emulator scripts are already public; the rest will be made public in due course, once things are reasonably stable.

