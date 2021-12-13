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

Going through each of these emails, one definitely stood out, and it was message_21. The others would be able to load and display the attached image/jpeg files; however this one wouldn't. Each email's attachments are also base64 encoded, which is pretty typical of email data, but it also leaves an attack vector to be exploited. Simply editing the eml header could hide a malicious script that can be base64 encoded as well. Trying to view this picture that won't load could potentially run a malicious script. 


[User's Emails]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task03/emails.zip
