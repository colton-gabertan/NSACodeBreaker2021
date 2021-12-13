# Task01 - Network Forensics, Command Line

**Task Instructions**:

The NSA Cybersecurity Collaboration Center has a mission to prevent and eradicate threats to the US Defense Industrial Base (DIB). Based on information sharing agreements with several DIB companies, we need to determine if any of those companies are communicating with the actor's infrastructure.

You have been provided a capture of data en route to the listening post as well as a list of DIB company IP ranges. Identify any IPs associated with the DIB that have communicated with the LP.

[capture] \
[ip ranges]

[capture]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task01/capture.pcap
[ip ranges]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task01/ip_ranges.txt

---

  This challenge is quite simple as we are given all the necessary information to start hunting for the machines that have communicated with the attacker's listening post. In order to determine the IP's that have done so, we need to take a look at the capture. I used my favorite network packet tool, WireShark.
