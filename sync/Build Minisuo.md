Minisuo utilizes cmake as the build-tool and can be built into both shared and static libraries.<br />Tongsuo-mini's build system depends on 'cmake' and it utilizes toolchain provided by Python for automated testing.
<a name="zLMx3"></a>
# Dependencies

- cmake 2.8.8 and higher
- python
   - pytest

The installation of the dependency is very different in various operating systems. This is a typical example on macOS as follows (based on homebrew):
```
brew install cmake
brew install python
sudo pip3 install -r test/requirements.txt
```
<a name="HzhbZ"></a>
# Build
Use the 'cmake' to build Tongsuo-mini. Run the following steps after Tongsuo-mini has been cloned into a local directory (inside that dir):
```
mkdir build
cd build
cmake ..
make
make test
```
