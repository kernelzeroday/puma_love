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




