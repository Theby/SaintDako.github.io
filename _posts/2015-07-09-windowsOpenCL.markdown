---
layout: post
title:  "Running OpenCL on Windows"
date:   2015-07-09 17:00:00
categories: windows, opencl
---

Sometimes, Windows can just be not-so-friendly for developers. I recently attempted to install and run some OpenCL code that I had written that ran on OS X / Linux, but it was not really working all that well. I eventually figured it out, and I decided to write this guide since it may be of some use to people.

For the record, I'm using Windows 7 (64-bit) and the GPU I'm using is the NVIDIA GTX 960. For a 32-bit machine, I believe [this guide](http://www.nvidia.com/Download/index.aspx?lang=en-us) should work just fine, but it didn't for me! I'll discuss how to compile the code using both g++ and Visual Studio.
<!-- more -->

If you are using a non-NVIDIA GPU, and have figured out how to run OpenCL code on Windows as well, then you should know that all of the posts on this blog are hosted on [the blog's GitHub page](https://github.com/SaintDako/SaintDako.github.io), so feel free to submit a pull request.

## NVIDIA GPUs

### Install ALL the things!

There are a few things to install. The first thing is the GeForce driver for your GPU, if you don't already have it (presumably you do). But if you don't, you can get it from [NVIDIA's Drivers page](http://www.nvidia.com/Download/index.aspx?lang=en-us).

After installing that, install CUDA (current version is 7.0) from [NVIDIA's CUDA Download Page](https://developer.nvidia.com/cuda-downloads). The network installer and the local installer are both fine - the former will download files during the installation, whereas the latter has everything bundled in one. Note that **CUDA comes with OpenCL 1.1**, which is quite old! I wouldn't hold my breath for a newer version of OpenCL, as NVIDIA GPUs use CUDA anyway.

Now, we need to install gcc (and g++). If you already have gcc, you need to make sure that it is the 64-bit version! If you installed MinGW-32, then it will be the 32-bit version, unfortunately. The way to get around this is by installing [the 64-bit version of MinGW, called MinGW-w64](http://mingw-w64.org/doku.php/download). I've only tested the *Mingw-builds* installer, as well as installing via Cygwin.


#### Mingw-builds

If using the mingw-builds installer, make sure to specify the correct settings when installing. I picked "win32" for the threads option.

After it installs, we need to add the bin to the PATH. I added the following directory to my PATH variable: `C:\Program Files\mingw-w64\x86_64-5.1.0-win32-seh-rt_v4-rev0\mingw64\bin`. Now in the command prompt, running `gcc --version` should result in something like `gcc <x86_64-win32-seh-rev0, Built by MinGW-W64 project> 5.1.0`.

#### Cygwin

To install via Cygwin, run the Cygwin set up file and search for `mingw64-x86-64`. I installed everything that comes up under the Devel category. Not all may be necessary, in fact only the gcc package probably is, but whatever.

In the Cygwin shell, running `gcc --version` will show us that g++ is indeed installed, but on my installation, this is not the 64-bit version of g++. The command is actually `x86_64-w64-mingw32-gcc`, which is terribly long, but it can be aliased.

### Compiling and running some code

Alright, now that everything should be installed (if your computer didn't kerplode), we can try compiling and running some stuff. Let's try finding all of the devices that support OpenCL on our computer by using the following C++ code:

{% highlight c++ %}
#include <iostream>
#include <CL/cl.hpp>

int main() {
    // get all platforms (drivers), e.g. NVIDIA
    std::vector<cl::Platform> all_platforms;
    cl::Platform::get(&all_platforms);

    if (all_platforms.size()==0) {
        std::cout<<" No platforms found. Check OpenCL installation!\n";
        exit(1);
    }
    cl::Platform default_platform=all_platforms[0];
    std::cout << "Using platform: "<<default_platform.getInfo<CL_PLATFORM_NAME>()<<"\n";

    // get default device (CPUs, GPUs) of the default platform
    std::vector<cl::Device> all_devices;
    default_platform.getDevices(CL_DEVICE_TYPE_ALL, &all_devices);
    if(all_devices.size()==0){
        std::cout<<" No devices found. Check OpenCL installation!\n";
        exit(1);
    }

    for (int i=0; i<all_devices.size(); i++)
        std::cout << "Device " << i << ": " << all_devices[i].getInfo<CL_DEVICE_NAME>() << "\n";

    exit(1);
}
{% endhighlight %}

I'll call this file `main.cpp` (original, I know).

#### Git Bash

The following two commands will create the object file and then the executable:

{% highlight bash %}
g++ -c -std=c++0x -IC:/Program\ Files/NVIDIA\ GPU\ Computing\ Toolkit/CUDA/v7.0/include  main.cpp  -o  main.o
g++ main.o -LC:/Program\ Files/NVIDIA\ GPU\ Computing\ Toolkit/CUDA/v7.0/lib/x64  -lOpenCL -o main.exe
{% endhighlight %}

or in one line:

{% highlight bash %}
g++ -std=c++0x -IC:/Program\ Files/NVIDIA\ GPU\ Computing\ Toolkit/CUDA/v7.0/include  main.cpp  -o  main.exe -LC:/Program\ Files/NVIDIA\ GPU\ Computing\ Toolkit/CUDA/v7.0/lib/x64 -lOpenCL
{% endhighlight %}

Yes, it's quite a bit to type, but that's why Makefiles exist!

#### Cygwin

The following two commands will create the object file and then the executable (note that we use `/cygwin/c/...` to access files on our C drive):

{% highlight bash %}
x86_64-w64-mingw32-g++ -c -std=c++0x -I/cygdrive/c/Program\ Files/NVIDIA\ GPU\ Computing\ Toolkit/CUDA/v7.0/include main.cpp -o main.o
x86_64-w64-mingw32-g++ main.o -L/cygdrive/c/Program\ Files/NVIDIA\ GPU\ Computing\ Toolkit/CUDA/v7.0/lib/x64 -lOpenCL -o main.exe
{% endhighlight %}

In one line:

{% highlight bash %}
x86_64-w64-mingw32-g++ -std=c++0x -I/cygdrive/c/Program\ Files/NVIDIA\ GPU\ Computing\ Toolkit/CUDA/v7.0/include main.cpp -o main.exe -L/cygdrive/c/Program\ Files/NVIDIA\ GPU\ Computing\ Toolkit/CUDA/v7.0/lib/x64 -lOpenCL
{% endhighlight %}

#### Command Prompt
Since Windows' command prompt is dumb, it looks a bit different from Git Bash. The following two commands will create the object file and then the executable:

{% highlight bash %}
g++ -c -std=c++0x -I"C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v7.0\include"  main.cpp  -o  main.o
g++ main.o -L"C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v7.0\lib\x64"  -lOpenCL -o main.exe
{% endhighlight %}

In one line:

{% highlight bash %}
g++ -std=c++0x -I"C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v7.0\include"  main.cpp  -o  main.exe -L"C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v7.0\lib\x64" -lOpenCL
{% endhighlight %}

## Conclusion

What a pain. Use Makefiles to *make* your life easier (get it?!) and have fun!
