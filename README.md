# Task06 - Reverse Engineering

**Task Instructions**:

Now that we've found a malicious artifact, the next step is to understand what it's doing. Identify some characteristics of the communications between the malicious artifact and the LP.

## Writeup

For this task, I feel that I took the quick and dirty approach, combining a bit of static and dynamic analysis to extract the necessary information needed to complete it. I did all of the analysis both on my windows machine as well as a docker containter built by the provided image from the last task.

Firstly, I had to load the image and spin up the container. I decided to debug within the container due to the fact that it held all of the dependencies necessary for the malware to run as intended. We can do so in a linux box with Docker via these commands:
```
docker load --input image.tar
docker run -it panic-nightly-test sh
```
> Be sure to include the sh at the end in order to gain the shell. If you omit it, it will spin up and crash due to the startup script failing.

