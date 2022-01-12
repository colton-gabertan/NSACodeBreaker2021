# Task05 - Docker Analysis

**Task Instructions**:

A forensic analysis of the server you identified reveals suspicious logons shortly after the malicious emails were sent. Looks like the actor moved deeper into OOPS' network. Yikes.

The server in question maintains OOPS' Docker image registry, which is populated with images created by OOPS clients. The images are all still there (phew!), but one of them has a recent modification date: an image created by the Prevention of Adversarial Network Intrusions Conglomerate (PANIC).

Due to the nature of PANIC's work, they have a close partnership with the FBI. They've also long been a target of both government and corporate espionage, and they invest heavily in security measures to prevent access to their proprietary information and source code.

The FBI, having previously worked with PANIC, have taken the lead in contacting them. The FBI notified PANIC of the potential compromise and reminded them to make a report to DC3. During conversations with PANIC, the FBI learned that the image in question is part of their nightly build and test pipeline. PANIC reported that nightly build and regression tests had been taking longer than usual, but they assumed it was due to resourcing constraints on OOPS' end. PANIC consented to OOPS providing FBI with a copy of the Docker image in question.

Analyze the provided Docker image and identify the actor's techniques.

*I would've provided the file, but the image.tar was too large as it contains entire filesystems*

> Enter the email of the PANIC employee who maintains the image
> ```
> ```
> 
> Enter the URL of the repository cloned when this image runs
> ```
> ```
> 
> Enter the full path to the malicious file present in the image
> ```
> ```

## Writeup

> This challenge was easing us into some malware analysis, setting us up nicely to start reverse engineering a binary. I wrote an article [here] as a very gentle introduction  into this topic if you've never done or seen anything of this nature

As we are provided with nothing but the image.tar, before loading it into Docker, I wanted to take a look at the strings of the folder before anything else. I was working on wsl, using the built-in strings command, then I stored the output to image.txt and dove in.

### Command to extract the strings
```
strings image.tar > image.txt
```

It returns an overwhelming amount of strings; however, we have some information from the flags we're looking for to help narrow the search. There is a repo that is cloned onto the container and the command to do so is: git clone. I searched for that in the text file and found a script that clones the repo, along with some of the commands run afterwards.

### Info found from image.txt
![image](https://user-images.githubusercontent.com/66766340/146481906-ee273079-7d06-4e77-beb8-6c1961c7aa63.png)

Scrolling up just a bit, I was also able to find who maintains this image, along with this employee's email.

### PANIC employee who maintains this image
![image](https://user-images.githubusercontent.com/66766340/146482069-c59e5eed-36e6-4a43-9182-662528cfb189.png)

At this point, all that was left was to look for the malicious binary somewhere within the filesystem. .tar is an extention that indicates compression, similar to a .zip, so in order to take a peek at the filesystem before loading it into Docker, we can simply unzip the image.tar and dive into the files.

In reference to the commands found after the github repo was cloned, I noticed something off with a few of them. Specifically the `make` command. From my experience coding with c++, makefiles are typically used to help with the compilation process of larger programs that have multiple dependencies. It allows us to compile one large program with the use of only one command, packaging it neatly together. For more information on makefiles and `make`, I recommend reading this [article].

### Unusual Make Command
![image](https://user-images.githubusercontent.com/66766340/146482533-498dfaca-7d10-41e8-b573-3512deb0936d.png)

After being tipped off by the unusual `make` commands, I decided to look for it in the filesystem and found it lurking in [/usr/bin/make].

![image](https://user-images.githubusercontent.com/66766340/146482842-f4b2a99b-ac1f-451c-a1ff-29739c199c82.png)

Given the nature of makefiles, I had never seen one quite that large and decided to take a closer look. My suspicion lead me to go straight to disassembling the binary, in true NSA fashion, I did so by popping it into [Ghidra]. Taking a look the decompilation of its main and some of the subroutines, we find a c++ program doing all sorts of malicious things.

![image](https://user-images.githubusercontent.com/66766340/146483267-accba269-7ff3-47c5-8da6-6732a5829c8f.png)

Sure enough, after a bit of basic static analysis, we've found a full-fledged program that does nothing what an actual make binary should be doing. Locating and disassembling this malware sets us up nicely to start reverse engineering it and getting to the bottom of what it does for the next task.

![image](https://user-images.githubusercontent.com/66766340/148634567-5c8aa635-1120-4f13-be30-bb42051ae8e9.png)

[here]: https://gabertan-colton.medium.com/practical-malware-analysis-basic-static-techniques-8897bd21b9e6
[article]: https://makefiletutorial.com/
[Ghidra]: https://ghidra-sre.org/
[/usr/bin/make]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task05/make
