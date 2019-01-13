# tf-windows-gpu
Tensorflow Nightly GPU is not being updated for Windows, so here's a repo with links to some compiled wheels I made myself.

## GUIDE ON BUILDING YOUR OWN TENSORFLOW ON WINDOWS WITH VS2017 ##
Follow the ordinary build instructions for building tensorflow from source on windows found here:
[https://www.tensorflow.org/install/source_windows](https://www.tensorflow.org/install/source_windows)

(You will need msys64 and all that jazz)

Make sure bazel is in your PATH variable and set BAZEL_VS to wherever your VS 2017 is installed 
BAZEL_VS=C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise

Run the `python configure.py` in the repo directory

Answer the questions as honestly as you can

Now open .tf_configure.bazelrc

Start the "x64 Native Tools Command Prompt" from Visual Studio, cause you will need variables from here.

Add the following lines in there:
```
build --action_env PATH="<COPY EVERYTHING IN YOUR PATH ENVIRONMENT FROM THE x64 NATIVE TOOLS COMMAND PROMPT HERE>"
build --copt=-DNOGDI --host_copt=-DNOGDI
build --action_env BAZEL_VS="<YOUR BAZEL VS ENVIRONMENT VARIABLE>"
```
Search the repo and find any reference to "Microsoft Visual Studio 14" and replace this to point to the relevant paths under your VS2017 install.

Then start the build... except...
I have Git installed (I think Visual Studio installed it) and this creates issues and complaints about cygwin errors because the different cygwin dll's between msys and git.

SO I USE THIS BATCH FILE TO START MY BUILD:
```
@ECHO OFF
cd z:\repo\github\tensorflow\tensorflow

IF EXIST "Z:\Program Files\Git" (
  mv "z:\Program Files\Git" "Z:\Program Files\Git_"
)

bazel --define=no_tensorflow_py_deps=true --config=opt //tensorflow/tools/pip_package:build_pip_package

IF EXIST "Z:\Program Files\Git_" (
  mv "Z:\Program Files\Git" "Z:\Program Files\Git_"
)
```

Now once the build has started and bazel starts compiling a few things, I KILL THE PROCESS WITH `CTRL-C`

Do a search in the folder created in your home directory `_bazel_username` (example: `C:\users\username\_bazel_username\`) for anything pointing to "Microsoft Visual Studio 14" and, again, replace it with the relevant paths to VS2017.

NOW BAZEL WILL ACTUALLY COMPILE TENSORFLOW FOR ME...

Also, there is an environment variable for BAZEL_LLVM but I don't think they've configured the toolchain for it so I couldn't get CLANG to work from LLVM instead.

__All are compiled with CUDA 10.0__  

| Date | Link | Python Version | TF API Version | CUDA Compat |
|------|------|----------------|----------------|-------------|
| 2019-01-12 | [tensorflow-cp37-cp37m-win_amd64.whl](https://1drv.ms/u/) * Still uploading * | Python 3.7.2 | v1 | 6.1 |
| 2019-01-08 | [tensorflow-cp37-cp37m-win_amd64.whl](https://1drv.ms/u/s!AiUbe609f8iritZu9BlMiHubpm0UCQ) | Python 3.7.2 | v1 | 6.1 |
| 2019-01-07 | [tensorflow-cp36-cp37m-win_amd64.whl](https://1drv.ms/u/s!AiUbe609f8iritZtpQtCf5k__Ad2Qg) | Python 3.6.6 | v1 | 6.1 |

### I REPEAT, THESE ARE COMPILED WITH CUDA 10.0 ###
Check your CUDA compatibility, I used 6.1 because I have a nVidia 1080 GTX Ti.

No CUDA 9.0 or 9.2.
Somewhere in the commits of Tensorflow they updated it to CUDA 10.0 and later version of cuDNN, so you may have to get these.

Also they are compiled for Python 3.6.6 and Python 3.7.2, but I may be dropping compiling for 3.6.6 soon since 3.7.2 seems a bit quicker.

These are compiled with /fp:fast, /arch:AVX, /Ox, /DNOGDI, etc.

They are also compiled using the later headers and libraries for MSVC 2017 (or should be at least) by changing the include path from the standard C:\Program Files (x86)\Microsoft Visual Studio 14.0\ to the 2017 Enterprise folder for Visual C++.

I suppose this means you may require the VS2017 VC++ redist instead of the 2015 VC++ redist.
I can't and won't test this, do it yourself.

I will chuck up a guide on how I got it to compile myself, since it required a bit of work FOR SOMETHING WHICH SHOULD FREAKING WORK OUT OF THE BOX... right?

I am still trying to see if I can manage to get XLA JIT to compile for Windows but it looks very unlikely.
Earlier commits were closer to getting XLA JIT to compile on windows than later commits were...
Of course, they keep changing things in there as they work on their XLA JIT so maybe it will eventually come good... somewhere over the rainbow...

Practically everything is two steps backwards, half a step forwards...

#### Note: Also BAZEL is a piece of sh!t, whoever invented that junk should re-evaluate their life... ####

Seriously, so many actual working build systems and they had to come up with a new one, which is:
* Unnecessarily difficult to even get to start building;
* Uses huge amounts of HDD space;
* Creates folders and symlinks to everywhere;
* Throws errors and hissy fits that are hard to diagnose;
* Slow;
* Uses hundreds of configuration files;
* Relies on Java (which _would_ be okay for portability if it didn't end up relying on native compilers to do all the work anyway);
* Vastly complex and indecipherable path, packages, etc.
* Wastefully downloads and builds components that are ALREADY INSTALLED ON THE MACHINE AND IN THE PATH (such as LLVM, Swig, etc.)
* Impossible to configure for toolchains and build systems... I mean why the hell would you make it so you have to create 3 different sections for a toolchain? Why not a folder with one configuration file per toolchain/compiler set?  
and the biggest issue:
#### BAZEL IGNORES THE CONFIGURATION FILES ANYWAY IT TURNS OUT, SINCE I CHANGED THEM ALL TO USE THE VS2017 DIRECTORY AND FILES BUT IN THE bazel-out and bazel-tensorflow directories it ignores the changes and goes back to VS2015 build tools ####

My suggestion for Google, drop Bazel completely or give it a total rethink and put it in the trash, shred the source code and pretend it never existed.
