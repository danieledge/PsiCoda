# How Coda is made

*17 November 2026.*

People have asked how any of this works, given there's no source code for the netBook's operating system. Here is the whole process, from Psion's own image to a card in the machine, and the setup I use to build and test it.

## Starting from Psion's image

The netBook doesn't keep its operating system in a chip. It loads it from the CF card at switch-on: a single file, OS.IMG, about 14 MB. Coda starts from Psion's last English release, v1.05(450), with the community c1.60a patch by Poke^Patisoners (2005), which already fixed a handful of Psion's own bugs: colour icons, the console's colours and so on.

That file is a ROM image: a tree of folders and files, laid out at fixed places in memory. The programs in it run straight from the ROM, where they sit, without being copied into RAM. That's what makes the job hard. Psion built each program to run at one exact address and kept no record of the addresses inside it, so normally nothing in the image can be moved.

## Taking things out

The first step is making room. Opera and Java came out (nearly 5 MB between them, and PsiWeb and PsiLinux do their jobs better now), along with Psion's old Email program, the plain FTP program, the old Comms terminal, the fax programs, which crashed without Email anyway, and a few viewers Word no longer needs.

Each removal is checked first. The build tool looks at what every remaining program imports and refuses to take out anything still in use, naming the program that needs it. Messaging stays, for example, because Contacts' "Send" uses it even with Email gone.

## Putting things in

The build tool reads the image's tree, takes things out, adds Coda's programs, pictures and settings, then writes the tree back. It keeps a few strict rules:

- **Psion's files stay exactly where they are.** Nothing Psion built is moved, so nothing breaks.
- **New files go in the gaps,** biggest first, each into the smallest gap it fits. A program has to be in one piece, and PsiWeb alone is 2 MB.
- **The folder lists are written fresh,** pointing at the new files, while the original lists the system still points into are left alone.
- **Nothing may overlap.** If any two files would share a byte, the build stops.

If it all doesn't fit, it drops the least important extras in a set order (spare wallpapers first, then extra typefaces) and tries again, and says what it left out.

Coda's programs are mostly built to run from the ROM too, so they take no RAM for their code. The bigger ones (PsiLinux, PsiPDF and others) live on the card instead and are installed or run from there.

## Room and memory

The ROM's size decides how much RAM you get: whatever the ROM doesn't use of the memory bank it's loaded into is RAM, in whole megabytes. On a 32 MB netBook a 14 MB ROM leaves about 18 MB. So freeing space inside the ROM gives Coda room for programs; it only gives you more RAM once the ROM's end drops past a megabyte boundary.

There's a way to go further. Psion released the same v1.05 software in several languages, and the same program sits at a different address in each. Comparing two builds word by word shows which words are addresses: a word that differs between them, and points to the same place in each, is an address. With five builds as witnesses that recovers the missing records for most of the image. Rebuilding a program at another build's address with them gives exactly Psion's bytes, every time (607 out of 607). It turns out someone did the same on the Series 5mx about twenty years ago. It works on the emulator as a prototype; it isn't in Coda's images yet.

## Changing Psion's programs without their source

There's no source code for Word, the System screen or the rest, so they're changed in four ways, from gentlest to deepest:

1. **Resources.** Menus, dialogs, texts and lists live in resource files, which can be edited and put back. The System screen's start list, the silkscreen icons and many of Coda's texts are done this way.
2. **Pictures and colours.** Icons, wallpapers and the colour schemes are just files in the image, replaced with new ones.
3. **Small byte changes.** Where a program does one wrong thing, the one instruction that does it is changed. Every change finds its target by its content, not by a fixed address, so it can't land in the wrong place.
4. **Hooks.** To add behaviour, one instruction in Psion's code becomes a jump to a tiny piece of code of ours. It does what the replaced instruction did, calls one of Coda's libraries, then carries on where it left off. That's how the toolbar gets its widgets and clock faces, how text gets smoothed, how themes are applied, and how Ctrl+Menu opens PsiHub. Each hook passes on anything it doesn't handle, so Psion's own behaviour is unchanged.

Every key and Command icon is checked afterwards against Psion's own image with the same script, so the shortcuts you know keep working.

## My setup

**The machine.** Everything is built on a Linux box with no special permissions, in a Docker image based on Ubuntu 24.04. It holds only the tools; the SDK sits outside it so builds are quick.

**The SDK.** Psion's EPOC Release 5 C++ SDK, the one Symbian shipped in 1999-2000, in the Linux packaging from the psion_cpp_sdk_linux project. Its compiler is Psion's GCC 3.0 (psion-98r2) targeting ARM, with Psion's binutils 2.9 (psion-98r2). Psion's own Windows tools from the SDK (the resource compiler, the bitmap converter, the installer maker) run under Wine 9.0. Everything is built for the ARMv4 instruction set of the netBook's StrongARM, and also runs on the Series 5mx's ARM710T.

**Other tools.** GCC 13.3 builds the host-side tests and tools. The image tools themselves (reading and writing ROM images, building the image and the card, checking keys) are written in TypeScript and run on Node 22. Pictures, icons and the boot screen are made with Python 3.12 and Pillow. The few tiny hook stubs are written in ARM assembly.

**The emulator.** Testing happens in the psionEmulators project (github.com/joehaines/psionEmulators), an emulator for Psion's machines, with its native test harness built with full optimisation. It boots the netBook's real bootloader with Coda's card, about 90 seconds of simulated time to the System screen, and can be driven from a script: taps on the screen, key presses (chords too), screenshots in colour, and a copy of the RAM to read files and logs back out. Every feature here was tested that way, in each colour theme, and each one gets its screenshots there: the pictures on this blog all come from it.

**The ROMs for testing** are Psion's own builds, kept with the emulator: the netBook's v1.05 in its several languages (also the witnesses for the address work above) and the Series 5mx's v1.05.

**What the emulator can't show.** It doesn't use the boot card as a second disk after starting, it doesn't model the netBook's passive colour screen, and it can't do networking, infrared or sound recording. So anything to do with the card, the real screen colours or the network gets its proper test on my own netBook, and the diary says which have.

## From build to netBook

A build produces the new OS.IMG, the bootloader's picture and a folder with everything else the card needs: installers, the programs that live on the card, and Psion's original image kept beside it. Copy that onto the card, switch on, and the netBook loads Coda. To go back, rename Psion's original to OS.IMG and take the battery out and back in.
