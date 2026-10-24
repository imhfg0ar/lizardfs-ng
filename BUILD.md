How to rebuild debian packages from the source provided.

If you wish to spend your time in vain, there is an instruction.

Download both *tar.xz files from the release page. The first one lizardfs-3.12.0+ng10-all.tar.xz should be unpacked to the build directory.
second - lizardfs_3.12.0+ng10-all.orig.tar.xz should be put one level up

cd to unpacked tree ( lizardfs-3.12.0+ng10-all/ if you use current build) and run `dpkg-buildpackage -us -uc`
if the build finishes normally - the ready-to-install deb's will be created one level up (same level as you put *orig* file)

Note you will SURELY unable to rebuild on anything other than ubuntu 18.

Feel free to play with debian/* if you wish, or start a bare source rebuild without proper packaging.

For source-only build: 
install build deps:
```
sudo apt-get install -y \
    cmake g++ make \
    libfuse-dev \
    libboost-filesystem-dev libboost-iostreams-dev \
    libboost-program-options-dev libboost-system-dev \
    zlib1g-dev libjudy-dev libsystemd-dev \
    libisal-dev \
    asciidoc unzip pkg-config
```

for debian or ubuntu. You will probably have no libjudy on modern systems, so skip it (the project can be build without it)


untar only `lizardfs_3.12.0+ng10-all.orig.tar.xz`
cd into unpacked tree - lizardfs-3.12.0+ng10-all
`mkdir build && cd build` # cmake will refuse to run from the sources tree itself

configure the build system:
```
cmake .. \
    -DCMAKE_BUILD_TYPE=RelWithDebInfo \
    -DCMAKE_INSTALL_LOCALSTATEDIR=/var \
    -DCMAKE_INSTALL_SYSCONFDIR=/etc \
    -DDEFAULT_USER=lizardfs \
    -DDEFAULT_GROUP=lizardfs \
    -DENABLE_CLIENT_LIB=ON \
    -DENABLE_POLONAISE=OFF \
    -DENABLE_DOCS=ON \
    -DENABLE_UTILS=ON \
    -DCMAKE_DISABLE_FIND_PACKAGE_PAM=TRUE \
    -DCMAKE_DISABLE_FIND_PACKAGE_DB=TRUE
```
DON'T play with the build settings. No user-servicable parts inside.
The only exception is the -DCMAKE_INSTALL_LOCALSTATEDIR=/var and -DCMAKE_INSTALL_SYSCONFDIR=/etc 
if you skip both - it will make full-local default paths, so the config goes to /usr/local/etc and data kept in /usr/local/var

If and only if you really want to replace already-installed debian/ubuntu packages, use
```            -DCMAKE_INSTALL_PREFIX="/"   \
               -DENABLE_DEBIAN_PATHS=YES    \
```
instead of both. The debian binaries WILL be overriden by make install. Configs should be safe but note I never run make install as root.
(the binary debs properly handle config files, never overriding already existing ones)

The second define changes 'mfs' in paths to lizardfs - debian's maintainer half-done work to co-exist with moosefs. Note it never worked and I never will fix!
