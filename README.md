# Task02 - Log Analysis

**Task Instructions**:

NSA notified FBI, which notified the potentially-compromised DIB Companies. The companies reported the compromise to the Defense Cyber Crime Center (DC3). One of them, Online Operations and Production Services (OOPS) requested FBI assistance. At the request of the FBI, we've agreed to partner with them in order to continue the investigation and understand the compromise.

OOPS is a cloud containerization provider that acts as a one-stop shop for hosting and launching all sorts of containers -- rkt, Docker, Hyper-V, and more. They have provided us with logs from their network proxy and domain controller that coincide with the time that their traffic to the cyber actor's listening post was captured.

Identify the logon ID of the user session that communicated with the malicious LP (i.e.: on the machine that sent the beacon *and* active at the time the beacon was sent).

* [Subnet associated with OOPS]
* [Network Proxy Logs from Bluecoat server]
* [Login data from domain controller]

## Writeup


[Subnet associated with OOPS]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task02/oops_subnet.txt
[Network Proxy Logs from Bluecoat server]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task02/proxy.log
[Login data from domain controller]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task02/logins.json
