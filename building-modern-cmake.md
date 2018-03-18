# How to build cpp-netlib with modern CMake on Windows with MSVC

## Step 1. Minimum requirements

- [CMake](https://cmake.org/download/) >= 3.11
- [OpenSSL](https://openssl.org/) (always the latest stable version)
- [Boost](http://www.boost.org/) >= 1.58
- [Microsoft Visual Studio](https://www.visualstudio.com/downloads/) >= 2015

## Step 2. Build dependencies

### OpenSSL (recommended)

OpenSSL building depends on:

- [Netwide assembler](http://www.nasm.us/)
- [Strawberry Perl](http://strawberryperl.com/) or ActiveState Perl, do NOT try msys2 perl, it won't work

You can install these applictation with help of [Chocolatey](https://chocolatey.org/)

```
choco install -y strawberryperl nasm
```

Download OpenSSL, e.g. with git

```
git clone https://github.com/openssl/openssl.git
cd openssl
git checkout OpenSSL_1_1_0g
```

Open Visual Studio command prompt and change directory to that with OpenSSL

```
set "PATH=%PATH%;C:\Program Files\nasm"
c:\Strawberry\perl\bin\perl.exe Configure VC-WIN32 --prefix=c:\OpenSSL-Win32
nmake
```

Open Visual Studio command prompt as administrator and change directory to that with OpenSSL

```
nmake install
```

You should have it installed into C:\OpenSSL-Win32 by now.

### Boost

Open Visual Studio command prompt and change directory to that with Boost sources

```
.\bootstrap --without-icu --with-libraries=chrono,log,program_options,date_time,thread,system,filesystem,regex,test
.\b2 toolset=msvc-14.1 link=static runtime-link=shared address-model=32 define=BOOST_USE_WINAPI_VERSION=0x0601 --build-type=complete
```

## Step 3. Build

Clone the repository

```
git clone --recursive https://github.com/btolfa/cpp-netlib
cd cpp-netlib
```

Configure with cmake

```
mkdir build
cd build
cmake .. -DCPP-NETLIB_BUILD_TESTS=ON -DCPP-NETLIB_BUILD_EXAMPLES=ON -DCPP-NETLIB_STATIC_BOOST=ON -DCPP-NETLIB_STATIC_OPENSSL=ON -DBOOST_ROOT=<path to boost> -DOPENSSL_ROOT_DIR=<path to opensll>
``` 

Build with cmake

```
cmake --build . --config Debug
```