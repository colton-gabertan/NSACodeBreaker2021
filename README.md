# Task06 - Reverse Engineering

**Task Instructions**:

Now that we've found a malicious artifact, the next step is to understand what it's doing. Identify some characteristics of the communications between the malicious artifact and the LP.

> Enter the IP of the LP that the malicious artifact sends data to
> ```
> ```
> 
> Enter the public key of the LP (hex encoded)
> ```
> ```
> 
> Enter the version number reported by the malware
> ```
> ```

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

Furthermore, I personally like to use the terminal user interface built into gdb as it can display the assembly, registers, and breakpoints as we step through the malware. I also like to view the assembly in intel syntax, as I'm more familiar with it. We can do this like so:
```
gdb make
set disassembly-flavor intel
layout asm
layout reg
```

#### GDB Setup
<img src="https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task06/task06-1.gif">

---

### Reversing

Luckily, this binary wasn't stripped of debug symbols, allowing us to successfully debug it without too many hiccups. There is even a well-defined `main()`, which is a good starting point to explore this program. Some of the subroutines are either mangled or have their names obfuscated, but that isn't much of a problem as we begin to walk through the execution of code and observe the disassembly as well as the decompilation.

If we run `start` in the debugger, it will take us to the entry-point of the program, a call to a subroutine named `gitGrabber()`. Before proceeding, let's take a look at this function in Ghidra.

### Starting Point
![image](https://user-images.githubusercontent.com/66766340/149032922-4809c818-cd10-4f40-8675-a98dd6e9a54d.png)

Off the bat, we can see a variable that potentially holds the IP of the LP, called `c++ ip`. Upon checking its XREFs (Cross-references), it gets initialized with 0x0 and defined after a call to `wcasbyllnvkwe()`. This function also took 0x13 as a parameter. We can infer that the return value of the function gets stored into `ip`, because return values typically get stored in `RAX`. And, then we can see a `MOV` of whatever gets stored into `RAX` to `ip`.

### `ip` Getting Defined
![image](https://user-images.githubusercontent.com/66766340/149034139-7b5191df-98ba-4533-8c74-ed0fd904b07e.png)

Checking out the decompilation of the `wcasbyllnvkwe()` function, we can see that its parameter is called `stringId`. It utilizes a switch-case statement to return a char* (string) based on the `stringId`, and there is even a case defined for the value of 0x13. From here, we can ensure that it is a safe bet that calling this function with 0x13 will show us whatever string gets stored in `ip` for `gitGrabber()` to use. Optionally, you can rename functions in Ghidra with `L`. I would use something like `getString`, but I opted out in renaming functions for this task.

### Decompilation for Case 0x13 - Potential IP
![image](https://user-images.githubusercontent.com/66766340/149035178-12263cd6-2864-4fec-b2e6-ea6a989aae7a.png)

Popping back into the debugger, we can run this function, passing it 0x13 as a parameter to see what gets stored in the `ip` variable.


