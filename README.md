# Task01 - Network Forensics, Command Line

**Task Instructions**:

The NSA Cybersecurity Collaboration Center has a mission to prevent and eradicate threats to the US Defense Industrial Base (DIB). Based on information sharing agreements with several DIB companies, we need to determine if any of those companies are communicating with the actor's infrastructure.

You have been provided a capture of data en route to the listening post as well as a list of DIB company IP ranges. Identify any IPs associated with the DIB that have communicated with the LP.

[capture] \
[ip ranges]

---

This challenge is quite simple as we are given all the necessary information to start hunting for the machines that have communicated with the attacker's listening post. In order to determine the IP's that have done so, we need to take a look at the capture. I used my favorite network packet tool, WireShark. Now, the tricky part is that we aren't given specific IP's, but we are given the ranges.

This means that we need to know about subnetting a network. Given the form of the range it looks a bit like **192.168.127.51/25**. This is known as CIDR notation, and it tells us which bits are reserved for the subnet and hosts, in turn telling us, which IP addresses actually exist in the subnet. For more information on how to actually calculate these ranges, I've written a separate article [here].

Upon opening the capture in wireshark, it can look overwhelming, as it displays all of the traffic in chronological order by default. However, to narrow it down, I decided to take a look at the endpoints.

**Wireshark Endpoints**
<img src="task01endpoints.gif" alt="Wireshark Endpoints">

[capture]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task01/capture.pcap
[ip ranges]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task01/ip_ranges.txt
[here]: https://gabertan-colton.medium.com/ipv4-subnetting-c2f70d772789
