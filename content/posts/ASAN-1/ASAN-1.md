---
date: '2026-01-14T01:22:40+02:00'
draft: false
title: 'Address Sanitizer (ASan): The What, Why, and How'
---

An Address Sanitizer, usually abbreviated as ASAN, is a runtime tool that enables discovering memory bugs. It was introduced by Google in 2012. And it is available for most modern compilers (LLVM, GCC, XCode, and MSVC).

It is useful to execute your code under a sanitizer while testing your code. It is a runtime tool (in contrast to static analysis tools), meaning that the code must be executed for the tool to be able to detect the memory bugs. 

ASAN can detect a the following classes of bugs:
1. Out-of-bounds accesses to heap, stack and globals
2.  Use-after-free
3. Use-after-return
4. Use-after-scope
5. Double-free
6. Memory leaks (still experimental in clang 20).

## How to add it to your current project ? 

I will be using docker in my example to point out to the main packages needed to use ASAN with clang. 

```Dockerfile
# Dockerfile
FROM ubuntu:24.04

RUN apt-get -y update && apt-get install -y --no-install-recommends \
 software-properties-common \
 gnupg \
 wget \
 nano \
 vim \
 bash-completion 

RUN wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key | apt-key add -

RUN apt-add-repository "deb http://apt.llvm.org/noble/ llvm-toolchain-noble-20 main"

RUN apt-get -y update && apt-get install -y --no-install-recommends \
clang-20 lldb-20 lld-20 libclang-rt-20-dev llvm-20 
```

In this Dockerfile, I am installing `software-properties-common` in order to be able to use apt-add-repository. [1](https://askubuntu.com/questions/593433/error-sudo-add-apt-repository-command-not-found)  `wget` is needed to fetch the `GPG key` for the llvm repo.  Then, we are adding the repository containing the packages for our ubuntu release.  We are also installing  `clang-20 lldb-20 lld-20` which are the compiler, the debugger, and the linker. then, we installed `libclang-rt-20-dev` which include the runtime library needed to run ASAN. And lastly, we are installing  `llvm-20` which includes the `llvm-symbolizer` which make the error msgs more readable.

## Hello, ASAN

The hello world example from ASAN is trying to use a variable that was already freed (Use-after-free).

```cpp
// example.cpp
int main(int argc, char **argv) {
  int *array = new int[100];
  delete [] array;
  return array[argc];  // BOOM
}
```

First of all, let's try compiling and executing the code without ASAN. 

```shell
clang++-20 -O1 -g example.cpp -o example_no_asan
```

To our surprise, nothing will happen. I totally get that this is a very anticlimactic code, yet this is a common behavior in subtle and not so subtle bugs. We would like to get feedback that something is wrong happened. And, that's why such a tool, like the one we are currently discussing, comes in handy, 

we can compile the code using the following command

```shell
clang++-20 -O1 -g -fsanitize=address -fno-omit-frame-pointer example.cpp -o example
```

Here we are using clang cpp compiler version 20.  The `-O1` flag is to apply minimum amount of code optimization. You can read more about compiler code generation flags from [here](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0). The `-g`
 flag is to generate debug information.  `-fsanitize=address` is the needed flag to execute the code with ASAN. `-fno-omit-frame-pointer` is used to get a nice stack trace as recommended by the [docs](https://releases.llvm.org/20.1.0/tools/clang/docs/AddressSanitizer.html#:~:text=To%20get%20nicer%20stack%20traces%20in%20error%20messages%20add%20%2Dfno%2Domit%2Dframe%2Dpointer.). 

When you execute the generated code `./example`, You will get a verbose (and actually useful) error msg telling you the first line the class of the detected error.

```shell
==290==ERROR: AddressSanitizer: heap-use-after-free on address 0x6dffe7ce0044 at pc 0x617e32210b4f bp 0x7ffd2ef30da0 sp 0x7ffd2ef30d98
```

and then, it will point out where it happened. in our case in `main`

```shell
previously allocated by thread T0 here:
    #0 0x617e3220f7a1 in operator new[](unsigned long) (/shared_folder/asan/example+0x1147a1) (BuildId: 46a719fff1df30113cf52892a234f902d414ceaf)
    #1 0x617e32210b12 in main /shared_folder/asan/example.cpp:2:16
    #2 0x70bfe880a1c9  (/lib/x86_64-linux-gnu/libc.so.6+0x2a1c9) (BuildId: 274eec488d230825a136fa9c4d85370fed7a0a5e)
    #3 0x70bfe880a28a in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x2a28a) (BuildId: 274eec488d230825a136fa9c4d85370fed7a0a5e)
    #4 0x617e32127354 in _start (/shared_folder/asan/example+0x2c354) (BuildId: 46a719fff1df30113cf52892a234f902d414ceaf)
```

and it will also give you a layout of the memory where the bug was detected. 

```shell
  0x6dffe7cdff00: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x6dffe7cdff80: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x6dffe7ce0000: fa fa fa fa fa fa fa fa[fd]fd fd fd fd fd fd fd
  0x6dffe7ce0080: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x6dffe7ce0100: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x6dffe7ce0180: fd fd fd fd fd fd fd fd fd fd fa fa fa fa fa fa
```

right after the layout, a legend explaining those symbols is also provided. in our case,  at the address (0x6dffe7ce0000) annotated by an arrow (=>), we tried to access a freed memory denoted by "fd". 

This concludes our little post. I hope it was helpful. :)

