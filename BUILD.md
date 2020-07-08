# Build with Microsoft Visual Studio 2019 with the Clang for Windows Subsystem for Linux Platform Toolset

#### 1) Setup your development environment
a) First, follow the instructions to install and setup WSL2 and Ubuntu 20.04 on Windows 10 here:

https://docs.microsoft.com/en-us/windows/wsl/install-win10

b) Once Ubuntu 20.04 is working, inside of Ubuntu's bash terminal install the following packages required for MSVS2019 WSL platform toolset support:
```
$ sudo apt-get install g++ gdb make ninja-build rsync zip
```

c) Install MSVS 2019 with the C++ for Linux Development component. For more information and instructions, see:

https://docs.microsoft.com/en-us/cpp/linux/download-install-and-setup-the-linux-development-workload?view=vs-2019

#### 2) Install Emscripten on Ubuntu 20.04 in WSL2
a) From your home directory in an Ubuntu bash terminal, get the Emscripten SDK and install the Emscripten "upstream" variant:
```
$ git clone https://github.com/emscripten-core/emsdk.git
$ ./emsdk/emsdk install latest-upstream
$ ./emsdk/emsdk activate latest-upstream
 ```
b) Enable Emscripten SDK tools on the command line for current shell
```
$ source ./emsdk/emsdk_env.sh
```
c) Make Emscripten SDK tools available on the command line for all WSL instances. Append the following using the editor of your choice to the bottom of your $HOME/.profile configuration file. Example:
```
$ nano $HOME/.profile
```
Copy-paste the the following to the end of .profile and save the file:
```
# import Emscripten development environment settings if it exists
if [ -f "$HOME/emsdk/emsdk_env.sh" ] ; then
    source $HOME/emsdk/emsdk_env.sh >/dev/null 2>&1
fi
```
Note that the stdout/stderr IO redirection above to /dev/null is necessary to suppress errors in Visual Studio when it starts a new WSL instance. 

#### 3) Build d3wasm
a) Inside of Microsoft Visual Studio 2019 or using Git for Windows, clone the d3wasm source code such that it is stored on the Windows file system outside of Ubuntu:
```
git clone https://github.com/jwtowner/d3wasm.git
```
b) With MSVS2019, open the *d3wasm/neo/d3wasm.sln* solution file and go to Main Menu -> Build -> Build All to build all projects. The build Configuration can be either Debug or Release, and the Platform either x86 or x64.

Normally, this should generate *d3wasm.html*, *d3wasm.js*, and *d3wasm.wasm* files in the bin/$(Configuration) output directory. However, due to a bug either in the MSVS2019 C++ for Linux build component or perhaps in WSL2, when the `MultiProcNumber` MSBuild property is set to a sufficiently high number to enable parallel building of translation units within a C++ project, the build may fail with the MSBuild Compile task throwing an unexpected exception from inside of the `liblinux.Local.Shell.WindowsSubsystemShell.VerifyEcho` member function. To complete the build, try building the solution a few times until everything succeeds or set the value of `MultiProcNumber` to a lower value inside of the *d3wasm/neo/Common.props* file. By default, this is set to the number of hardware threads available on the CPU of the build machine, using the `$([System.Environment]::ProcessorCount)` MSBuild property.

#### 4) Package the game demo data
a) Get the Doom 3 Demo from Fileplanet (or other sources): https://www.fileplanet.com/archive/p-15998/DOOM-3-Demo

b) Install the Demo (using wine on Linux)

c) Copy __<D3Demo_install_path>/demo/demo00.pk4__ to __<D3wasm_path>/build-wasm/data/demo__ folder

d) Package the data using the *package_demo_data* cmake target
```
make package_demo_data
```
Normally, this should have generated *demo00.data* and *demo00.js* files

#### 5) Deploy locally
a) Run a local web server from the build directory. The simplest is to use python SimpleHTTPServer:
```
python -m SimpleHTTPServer 8080
```
or (python 3)
```
python -m http.server 8080
```
b) open your favorite Browser to http://localhost:8080/d3wasm.html

#### 6) Enjoy!

**NB**: it is possible to do a native build also, to compare the performance for example. 
To do so:
```
  mkdir build-native
  cd build-native
  cmake ../neo -DCMAKE_BUILD_TYPE=Release
  make
```
Packaging the data works differently for native builds. Have a look at https://github.com/dhewm/dhewm3/wiki/FAQ
