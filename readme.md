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



