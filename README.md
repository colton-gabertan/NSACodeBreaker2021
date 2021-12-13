# Task04 - Powershell, Registry Analysis

**Task Instructions**:

A number of OOPS employees fell victim to the same attack, and we need to figure out what's been compromised! Examine the malware more closely to understand what it's doing. Then, use these artifacts to determine which account on the OOPS network has been compromised.

* [OOPS forensic artifacts]

## Writeup

After examining the downloaded powershell script from the task03, it is clear that it scrapes SCP and SSH session keys. This is most likely so that the attacker can escalate further into OOPS's network and compromise more machines. Based on the artifacts provided, we have public keys, private keys, and a snapshot of the machine's registry. The first thing we would do is take a look at the artifacts and draw some conclusions based on the state of them.

Public key data isn't too important as, by its name, is generally safe for anyone to have; however, private keys, which are used to decrypt data encrypted by public keys are a bit more sensite. So, lets take a look at those for now. The private keys are marked with the file extension .ppk.

### Viewing .ppk data
<img src="https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task04/artifacts.zip">

A quick glance reveals that there is no encryption for a couple of the private keys, meaning there's no password required to use them. Now that we know that there are unsafe private keys thrown in with these artifacts, all that's left is to see which sessions were active during the time of the breach, and which ones potentially got stolen. It is bad news if one of the unencrypted keys was in use while the malicious script ran.

I like to use a tool called [WRR] as it can display and parse NTUSER.DAT files very well. To narrow down the scope of the search, we need to take another look at the malicious [script]. Here we can view where it traveled to within the registry, and exactly what it was looking for. There are clear paths defined within the script here:

```powershell
$PuTTYPathEnding = "\SOFTWARE\SimonTatham\PuTTY\Sessions"
$WinSCPPathEnding = "\SOFTWARE\Martin Prikryl\WinSCP 2\Sessions"
```


[OOPS forensic artifacts]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task04/artifacts.zip
[WRR]: https://www.mitec.cz/wrr.html
[script]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task03/zdfou.sh
