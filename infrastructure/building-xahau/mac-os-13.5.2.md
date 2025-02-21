# Mac OS - 15.0.1

| Dependency  | Working Version |
| ----------- | --------------- |
| Apple Clang | 14.3.1          |
| LLVM        | 14              |
| LLD         | 14              |
| Boost       | 1.86.0          |
| CMake       | 3.23.1          |
| Protobuf    | 3.20.0          |
| WasmEdge    | 0.11.2          |

### Force Apple Clang 14.3.1

1. Download an older version of Xcode
   1. Go to the [https://developer.apple.com/download/more/](https://developer.apple.com/download/more/) page. You will need to sign in with your Apple Developer account.
   2. Search for the version of Xcode that includes Apple Clang 14. This is typically specified in the release notes for each Xcode version.
   3. Download the Xcode \`.xip\` file for the version you need.
2. Install the older version of Xcode:
   1. Once the download is complete, extract the \`.xip\` file, which will give you an Xcode application.
   2. Rename this Xcode application if you want to keep multiple versions of Xcode on your system (e.g., \`Xcode\_14.3.1.app\`).
   3. Drag the Xcode application to your \`/Applications\` directory.
3. Switch to the desired version of the toolchain
   1.  If you want to use the newly installed version of Xcode and its toolchain by default, you can switch to it using the \`xcode-select\` command:

       ```
       sudo xcode-select -s /Applications/Xcode_14.3.1.app/Contents/Developer
       ```
   2. Replace \`Xcode\_14.3.1.app\` with the actual name of the Xcode version you installed.
   3.  Open Terminal and run the following command to check the version of Clang:

       `clang --version`

{% hint style="info" %}
If you want to use a specific version of Apple Clang for command line builds you can use the following;
{% endhint %}

```
export DEVELOPER_DIR=/Applications/Xcode_14.3.1.app/Contents/Developer
clang --version
```

### Set Build Env Variables

First make a dependency directory. I like to use `~/dependencies`

```
mkdir ~/dependencies
```

Next we need to set the environment variables.

### Set Version Env Variables

```
export LLVM_VERSION=14
export BOOST_VERSION=1.86.0
export WASMEDGE_VERSION=0.11.2
export PROTOBUF_VERSION=3.20.0
```

### Set Build Env Variables

```
export DEP_DIR=~/dependencies
export BOOST_FOLDER_NAME="boost_$(echo "$BOOST_VERSION" | sed 's/\./_/g')"
export BOOST_ROOT="$DEP_DIR/$BOOST_FOLDER_NAME"
export BOOST_LIBRARY_DIRS="$BOOST_ROOT/libs"
export BOOST_INCLUDE_DIR="$BOOST_ROOT/boost"
export BOOST_CXXFLAGS="${BOOST_CXXFLAGS:-} -DBOOST_ASIO_HAS_STD_INVOKE_RESULT"
export LLVM_PREFIX=`brew --prefix llvm@$LLVM_VERSION`
export LLVM_DIR="$LLVM_PREFIX/lib/cmake/llvm"
export LLVM_LIBRARY_DIR="$LLVM_PREFIX/lib"
export LLD_DIR="$LLVM_PREFIX/lib/cmake/lld"
export CC=clang
export CXX=clang++
export LDFLAGS="${LDFLAGS:-} -L$BOOST_LIBRARY_DIRS -L$LLVM_PREFIX/lib"
export CPPFLAGS="${CPPFLAGS:-} -I$BOOST_ROOT/include -I$LLVM_PREFIX/include"
export CFLAGS="${CFLAGS:-} -DBOOST_ASIO_HAS_STD_INVOKE_RESULT"
export CXXFLAGS="${CXXFLAGS:-} -DBOOST_ASIO_HAS_STD_INVOKE_RESULT"
```

### Install Core Dependencies

```
brew update
brew install automake \
             wget \
             curl \
             git \
             pkg-config \
             openssl \
             autoconf \
             libtool \
             unzip \
             cmake \
             "llvm@$LLVM_VERSION"
```

### Install Rippled Dependencies

Install Protobuf

```
cd $DEP_DIR && \
wget -nc https://github.com/protocolbuffers/protobuf/releases/download/v$PROTOBUF_VERSION/protobuf-all-$PROTOBUF_VERSION.tar.gz && \
tar -xzf protobuf-all-$PROTOBUF_VERSION.tar.gz && \
cd protobuf-$PROTOBUF_VERSION/ && \
./autogen.sh && \
./configure --disable-shared link=static && \
make -j$(sysctl -n hw.logicalcpu) && \
sudo make install
```

Install Boost

```
cd $DEP_DIR && \
wget https://archives.boost.io/release/$BOOST_VERSION/source/$BOOST_FOLDER_NAME.tar.gz && \
tar -xvzf $BOOST_FOLDER_NAME.tar.gz && \
cd $BOOST_FOLDER_NAME && \
./bootstrap.sh && \
./b2 -j$(sysctl -n hw.logicalcpu)
```

Install WasmEdge

```
# You will need to edit the CMakesList.txt to update the boost dependency url to use a new prefix
# "https://archives.boost.io/release/" due to https://github.com/boostorg/boost/issues/843#issuecomment-2567034014
cd $DEP_DIR && \
wget -nc -q https://github.com/WasmEdge/WasmEdge/archive/refs/tags/$WASMEDGE_VERSION.zip && \
unzip -o $WASMEDGE_VERSION.zip && \
cd WasmEdge-$WASMEDGE_VERSION && \
mkdir build && \
cd build && \
cmake -DCMAKE_BUILD_TYPE=Release \
  -DWASMEDGE_BUILD_SHARED_LIB=OFF \
  -DWASMEDGE_BUILD_STATIC_LIB=ON \
  -DWASMEDGE_BUILD_AOT_RUNTIME=ON \
  -DWASMEDGE_FORCE_DISABLE_LTO=ON \
  -DCMAKE_POSITION_INDEPENDENT_CODE=ON \
  -DWASMEDGE_LINK_LLVM_STATIC=ON \
  -DWASMEDGE_BUILD_PLUGINS=OFF \
  -DWASMEDGE_LINK_TOOLS_STATIC=ON \
  .. && \
make -j$(sysctl -n hw.logicalcpu) && \
sudo make install
```

### Clone the repository

```
mkdir ~/projects && \
cd ~/projects && \
git clone https://github.com/Xahau/xahaud.git && \
cd xahaud && \
git checkout dev
```

### Build Xahaud

From the root `xahaud` directory:

```shellscript
mkdir build && \
cd build && \
cmake -DCMAKE_BUILD_TYPE=Debug -DLLVM_DIR=$LLVM_DIR -DLLVM_LIBRARY_DIR=$LLVM_LIBRARY_DIR .. && \
cmake --build . --target rippled --parallel -j$(sysctl -n hw.logicalcpu)
```

Start the built node

```
./rippled
```
