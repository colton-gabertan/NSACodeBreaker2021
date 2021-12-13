# Task02 - Log Analysis

**Task Instructions**:

NSA notified FBI, which notified the potentially-compromised DIB Companies. The companies reported the compromise to the Defense Cyber Crime Center (DC3). One of them, Online Operations and Production Services (OOPS) requested FBI assistance. At the request of the FBI, we've agreed to partner with them in order to continue the investigation and understand the compromise.

OOPS is a cloud containerization provider that acts as a one-stop shop for hosting and launching all sorts of containers -- rkt, Docker, Hyper-V, and more. They have provided us with logs from their network proxy and domain controller that coincide with the time that their traffic to the cyber actor's listening post was captured.

Identify the logon ID of the user session that communicated with the malicious LP (i.e.: on the machine that sent the beacon *and* active at the time the beacon was sent).

* [Subnet associated with OOPS]
* [Network Proxy Logs from Bluecoat server]
* [Login data from domain controller]

## Writeup

To start off, the subnet associated with OOPS is a bit misleading. The logs we are looking at are proxied through the bluecoat server, meaning the given IP range won't really help us, as the traffic is run through the proxy and not the machine itself. Therefore, the approach we would need to take is to try to match up the times of the machines that were active while the beacon was sent. However, the subnet does help us narrow down our search for the needle in the haystack if we refer back to the pcap capture from task 1.

I went out on a whim, given that we are looking at a lot of internet traffic, and decided to look at the http information of the pcap. Sure enough, we can see some communication between a shady URI *http://zdfou.invalid* and a machine from OOPS's network.

### OOPS Traffic
<img src="https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task02/subnetTraffic.jpg">

A quick search of the shady URI in the proxy log yielded us a result.

### OOPS machine traffic on the proxy log
<img src="https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task02/zdfou.jpg"
     
If we take a closer look at the times, we can see that the pcap has a time of 12

[Subnet associated with OOPS]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task02/oops_subnet.txt
[Network Proxy Logs from Bluecoat server]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task02/proxy.log
[Login data from domain controller]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task02/logins.json
