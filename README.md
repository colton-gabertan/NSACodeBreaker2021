# Task06 - Reverse Engineering

**Task Instructions**:

Now that we've found a malicious artifact, the next step is to understand what it's doing. Identify some characteristics of the communications between the malicious artifact and the LP.

## Writeup

For this task, I feel that I took the quick and dirty approach, combining a bit of static and dynamic analysis to extract the necessary information needed to complete it. I did all of the analysis both on my windows machine as well as a docker containter built by the provided image from the last task.

### Setting Up the Environment

Firstly, I had to load the image and spin up the container. I decided to debug within the container due to the fact that it held all of the dependencies necessary for the malware to run as intended. We can do so in a linux box with Docker via these commands:
```
docker load --input image.tar
docker run -it panic-nightly-test sh
```
> Be sure to include the `sh` at the end in order to gain the shell. If you omit it, it will spin up and crash due to the startup script failing.

Now that we've got the container up and running, we can see that it's Alpine linux, meaning in order to get our tools, we need to make use of the `apk` command. I took on this challenge with the bare bones `gdb`, seeing that it was a c++ binary in a linux environment.
```
apk update
apk add gdb
```

From here, we just need to navigate to the malicious `make` in `/usr/bin` and start debugging. Simultaneously, I had my other copy of `make` open in Ghidra, so I could follow along with the execution of code.

Furthermore, I personally like to use the terminal user interface built into gdb as it can display the assembly, registers, and breakpoints as we step through the malware. I also like to view the assembly in intel syntax, as I'm more familiar with it. We can do so like so:
```
gdb make
set disassembly-flavor intel
layout asm
layout reg
```

### GDB Setup
<img src="https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task06/task06-1.gif">
