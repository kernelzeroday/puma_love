some notes about a puma engagement

10.0 -> 10.1 -> 10.1.5 -> 10.1 developer tools cd


SSH CLIENT CONFIGURATION:

Host g3
  HostName 10.0.0.245
    Ciphers 3des-cbc
    KexAlgorithms +diffie-hellman-group1-sha1
    HostKeyAlgorithms=+ssh-dss
    User kelsey

ssh g3

Do not ssh to the IP



bash 205b
(ins)kod@m1:~/Downloads/APPLE/apple_open$ wget http://ftp.gnu.org/gnu/bash/bash-2.05b.tar.gz

sudo cp /bin/sh /bin/sh.apple
sudo cp /usr/local/bin/bash /bin/sh

[localhost:~/ports/pkgsrc/bootstrap] kelsey% sudo ./bootstrap --full

# NO GO

move apple sh back

=============


Attempt build cctools and apple gcc

(ins)kod@m1:~/Downloads/APPLE/apple_open$ wget https://github.com/apple-oss-distributions/cctools/archive/cctools-384.tar.gz

(ins)kod@m1:~/Downloads/APPLE/apple_open$ wget https://github.com/apple-oss-distributions/gcc/archive/gcc-932.1.tar.gz


/usr/local/lib does not exist, make it and subdir system

cryptic response about no streams header. lets try the only match on the apple open source page for "streams", which proclaims itself a library.

we get clean build
we get clean install

how fascinating, a new error. lets try more lib builds
images.c:64: header file 'architecture/ppc/processor_facilities.h' not found


libsystem first
very no go


https://github.com/apple-oss-distributions/architecture/archive/architecture-229.tar.gz
lets try


bruteforce wins, downthemall to collect every link on apple foss page, pull all, push sftp to g3, extract all, control f for processor_facilities.h

its in libc

clean build
clean install


manually move /System/Library/Frameworks/System.framework/Versions/B/PrivateHeaders/architecture

to /System/Library/Frameworks/System.framework/PrivateHeaders


success


indr issues with 10.1.5 cctools

attempt build 10.0 cctools

/usr/bin/ld: can't locate file for: -lcrt0.o


lets attempt an older Csu


well i'll be darned. 

https://opensource.apple.com/source/gcc/gcc-1495/README.Apple

PREREQUISITES

Presumably if you're reading this, you've figured out how to get the
sources. :-) But just to be complete, these sources are available from
the Darwin repository at opensource.apple.com, CVS module "gcc3".  See
http://www.opensource.apple.com/tools/cvs if this isn't enough info
yet.

If you want C++ exception handling to work, you will need a modified
crt1.o. (crt1.o is the bit of code that sets up for execution and
calls your program's main().) The modified crt1.o is standard in 10.2,
but 10.1, you will need to set it up yourself.

If you can't get a modified crt1.o from somebody else, you can patch a
copy of the sources to the "Csu" project and build it yourself.  The
patch is included in this directory, as "csu-patch".  The build is
easy, just say "make" in the Csu directory, and then copy the crt1.o
to /usr/lib/crt1.o (as usual, it's prudent to keep around a copy of
the original crt1.o, just in case).  You will need to have built the
"cctools" project as well, in order to get the helper tool "indr"
(which is expected to be installed as /usr/local/bin/indr).

BUILDING, THE APPLE WAY

To build things the Apple way, just say (in the source directory)

	mkdir -p build/obj build/dst build/sym
	gnumake install RC_OS=macos RC_ARCHS=ppc TARGETS=ppc \
		SRCROOT=`pwd` OBJROOT=`pwd`/build/obj \
		DSTROOT=`pwd`/build/dst SYMROOT=`pwd`/build/sym

This will configure and then do a full bootstrap build, with all the
results going into the subdirectory build/ that you created.  The
final results will be in the "dest root" directory build/dst, in the
form of an image of the installed directory structure.  The drivers
and other user-visible tools have a "3" suffixed, so for instance the
driver is /usr/bin/gcc3, and the demangler is /usr/bin/c++filt3.

To install the results, become root and do

	ditto build/dst /

Various knobs and switches are available, but even so, the Apple
makefile machinery is mainly designed for mass builds of all the
projects that make up Darwin and/or Mac OS X, and is thus not as
flexible as the standard GCC build process.

To build for i386 Darwin, set TARGETS=i386.  To build fat, set
RC_ARCHS='i386 ppc' TARGETS='i386 ppc'.  Note that you must have a
complete set of fat libraries and i386-targeting cctools for this
all to work.

You can set the four *ROOT variables to point anywhere, but they must
always be absolute pathnames.

This way of building may or may not work on non-Macs, and if it
doesn't, you're on your own.




https://opensource.apple.com/source/gcc3/gcc3-1161/csu-patch


digging reveals 

https://opensource.apple.com/source/cctools/cctools-384.1/

a tarball is found on

http://src.gnu-darwin.org/DarwinSourceArchive/apsl/

http://src.gnu-darwin.org/DarwinSourceArchive/apsl/cctools-384.1.tar.gz


it looks more promising.

watching it build.


we receive the same error. 


going on a binary hunt.
https://gist.github.com/theevilbit/906149b8e7273cee5df41a2d5f68ba48


downloading darwin isos looking for elusive object files crt0.o and crt1.o
a binary mach-0 for indr would work too


warning here be dragons
https://ia800600.us.archive.org/19/items/v6-manual/v6-manual.pdf

our research into crt0 has lead us down dark paths into unix v6 manual. a bad omen

"The diagnostics produced by C itself are intended to be self-explanatory. Occasional messages may be pro-
duced by the assembler or loader. Of these, the most mystifying are from the assembler, in particular ‘‘m,’’
which means a multiply-defined external symbol (function or data)."
        -- some ancient sage

fantastic...




====


could it be that i was a fool? blind like a child, unable to see clearly in the simplicity of it all?

I maybe forgot to install dev tools for 10.1.

I could be coasting on a semi complete development environment, hence the missing crt0.o which the documentation insists should exists, which builds depend on. Perhaps it is simple.

I shall test.


=-=-=-=-=-=-

jan 10 2023


some progress has been made, albeit slowly.


This document explains some otherwise hidden details about build order and build execution!
https://www-jlc.kek.jp/~fujiik/macosx/memo/HEPonX.html

Using this document I was able to begin to make progress on building cctools

I need to fix blockers on the compliation of the xnu kernel, to use the headers, to build cctools!


this document may hold some secrets about compliling XNU 

https://theswissbay.ch/pdf/Gentoomen%20Library/Programming/Misc/Manning.Publications.Company.Programming.Mac.OS.X.A.Guide.for.Unix.Developers.eBook-DDU.pdf

> Chapter 8 introduces Jaguar, Apple's most recent Mac OS X release. It pre- ... May 30 14:51:26 PDT 2002; root:xnu/xnu-201.42.3.obj~1/RELEASE_PPC Power.



I have come to believe that the CC compiler included with devtools MAY BE the mysterious "gcc 2.95.2" that supposedly exists for 10.1 devtools.
I am very much not certain about this and need to do much deeper investigation.
The man page for cc strongly suggests it is not, however /usr/libexec/ contains some gcc 2.95.2 files and cpp, which it claims comes from gcc.
If this is the case, and cc is simply gcc 2.95.2 with signifigant changes to enable objective c, then really now its just getting XNU to not hate me during the compile process.

Fink documentation seems to support this assertion.
> Short story: The compiler is a gcc derivate, but installed as cc; you may have to patch Makefiles. Most packages won't build shared libraries. If you get errors related to macros, use the -no-cpp-precomp option.

Long story: The compiler tool chain in the Mac OS X Developer Tools is a strange beast. The compiler is based on the gcc 2.95.2 suite, with modifications to support the Objective C language and some Darwin quirks. The preprocessor (cpp) is available in two versions. One is the standard precompiler (from gcc 2.95.2), the other one is a special precompiler written by Apple, with support for precompiled headers. The latter one is used by default, because it is faster. However, some code doesn't compile with Apple's precompiler, so you must use the -no-cpp-precomp option to get the standard precompiler. (Note: I previously recommended the -traditional-cpp option. The semantics of this option have changed slightly with GCC 3, breaking most packages that use it. -no-cpp-precomp has the desired effect on both the current Developer Tools and future compilers based on GCC 3.)

The assembler says it's based on gas 1.38. The linker is not based on GNU tools. This is a problem when building shared libraries, as GNU libtool and configure scripts generated by it don't know how to handle Apple's linker.


https://www.finkproject.org/doc/porting/porting.en.html


Various other places online seem to make mention of this.
https://www.nntp.perl.org/group/perl.macosx/2002/12/msg4431.html


This link claims to contain an april 2002 devcd for 10.1 however on extraction it seems to just be a december cd.
https://www.macintoshrepository.org/26017-apple-developer-connection-2002-

I need to download and extract more cd images. 


Searching for crt0.o using find on the 10.2 cds was also fruitless. I may need to install in qemu and hunt the live system.



TRY THIS NEXT TIME:
mkdir -p build/obj build/dst build/sym
gnumake install RC_OS=macos RC_ARCHS=ppc TARGETS=ppc \
SRCROOT=`pwd` OBJROOT=`pwd`/build/obj \
DSTROOT=`pwd`/build/dst SYMROOT=`pwd`/build/sym



this eliminated 2 errors in 
bash-2.05b$ cp -r /usr/include/mach EXTERNAL_HEADERS
bash-2.05b$ make exporthdrs




==--==--==--==--==--==

It is my birthday today.

I have slain 2 dragons. crt0.o and indr lay at my feet, bleeding.

Reading again my cctools error i decided to check if otool already existed on my system, the utility blocking my build. It did, so I thought to try again to bypass and build indr directly. 
I had previously attempted this but foolishly did not closely inspect the directory after build.
indr.NEW was a valid ppc executable sitting right there! 
When i attempted to run it however i was greeted with a familiar Illegal Instruction error.have

Using a copy of cctools-384.1 i was able to build a WORKING copy of indr.NEW and manually placed it into my path.

Csu built my fabled crt0.o with the gcc patch applied.


I now face libm as a foe, however on inspection apple never released a libm source file.

Some digging on ancient message posts revealed the libm implementation is inside Libsystem.

Libsystem needs a few things like ncurses which i had previously built and LibInfo among other .a files.


New dragons fly above me and circle



