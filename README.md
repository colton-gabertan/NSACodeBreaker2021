# Task03 - Email Analysis

**Task Instructions**:

With the provided information, OOPS was quickly able to identify the employee associated with the account. During the incident response interview, the user mentioned that they would have been checking email around the time that the communication occurred. They don't remember anything particularly weird from earlier, but it was a few weeks back, so they're not sure. OOPS has provided a subset of the user's inbox from the day of the communication.

Identify the message ID of the malicious email and the targeted server.

* [User's Emails]

## Writeup

In a realistic environment, I would advise to never blatantly open the emails on a baremetal workstation. Instead, this is the point where we should spin up a virtual machine tailored for malware analysis. This will protect us from accidentally running or downloading some malware. However, given that this is a CTF you *should* be fine to do so and observe the behavior of the emails. 

With email analysis, simply viewing the emails with a typical .eml viewer or browser will not show us the email header and any other hidden data. So, I decided to check them out in sublime text, as well as using a typical .eml viewer that comes default with Windows 10. It is important to get the view of what the user saw, but more so what we can find by taking a closer look.

### To demonstrate
<img src="https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task03/task03eml.gif">


[User's Emails]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task03/emails.zip
