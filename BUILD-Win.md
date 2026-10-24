How to compile windows driver

My buildhost is Ubuntu24 with mingw-w64. Note I never tried to natively build on windows.

## 1. install mingw and binutils-mingw 
you need pthread model, not the windows-native
`x86_64-w64-mingw32-g++ -v` should return Thread model posix

## 2. Build Boost 1.83 for MinGW
(you will need bison if you build from ubuntu source - it has one key file stripped out)
`apt source libboost1.83-dev ` # real package, not unversioned compatibility shim
```
cd boost1.83-1.83.0
(cd tools/build/src/engine && bison --defines=jamgram.hpp -o jamgram.cpp jamgram.y)
./bootstrap.sh --with-toolset=gcc
cat > user-config.jam << 'JAMEOF'
using gcc : mingw : x86_64-w64-mingw32-g++ :
  <rc>x86_64-w64-mingw32-windres
  <archiver>x86_64-w64-mingw32-ar
  <ranlib>x86_64-w64-mingw32-ranlib
  ;
JAMEOF
./b2 --user-config=user-config.jam \
  toolset=gcc-mingw target-os=windows address-model=64 \
  variant=release link=static threading=multi \
  --with-filesystem --with-iostreams --with-program_options --with-system \
  --prefix=$BOOST install -j4

```
(of course, don't install it system-wide, /tmp/boost should fit as BOOST)

## 3. windows build itself

unpack winfsp-ng.tar.gz archive from release page to separate directory, cd inside,
```
mkdir -p build-win && cd build-win
cmake .. \
  -DCMAKE_TOOLCHAIN_FILE=../cmake/mingw32-w64-toolchain.cmake \
  -DBOOST_ROOT=$BOOST -DBoost_NO_SYSTEM_PATHS=ON \
  -DENABLE_POLONAISE=OFF -DENABLE_DOCS=OFF \
  -DCMAKE_DISABLE_FIND_PACKAGE_DB=TRUE -DCMAKE_DISABLE_FIND_PACKAGE_PAM=TRUE \
  -DCMAKE_DISABLE_FIND_PACKAGE_Judy=TRUE -DCMAKE_DISABLE_FIND_PACKAGE_Thrift=TRUE \
  -DCMAKE_DISABLE_FIND_PACKAGE_Polonaise=TRUE \
  -DCMAKE_BUILD_TYPE=Release
make
```

you will find `build-win/src/mount/fuse/mfsmount.exe` if it completes properly
