# NTFS Alternate Data Streams

![yo_dawg](https://github.com/aalex954/ADS-Example/assets/6628565/47a7a6e5-cb3c-42fc-972e-bfe12e6bed58)

## Background

Alternative NTFS streams or Alternative Data Streams are a feature of NTFS file system. ADS was originally designed to provide compatibility (resource forks) with Macs which used the Hierarchical File System (HFS). 
ADS is effectively used to store additional information (meta data) in a file or directory. Legitimate uses of ADS include, zone information (Mark of the Web), encryption, AV optimizations, and SQL database integrity. There are also malicious uses for ADS including hiding malicious code in existing files. 

notes: 
- ADS data will be lost if a file or directory is moved to a non-NTFS file system.
- ADS cannot be disabled.

## Abuse Potential

[SANS ADS](http://www.uqac.ca/flemieux/INF341/NTFS_Stream.pdf)

### Signed Malware

When signing a file, if the signing application does not also target the alternate data streams, hidden malware may become more trusted.

---

### Hiding Malicious Payloads in Files or Directories

Given the inability of most Windows applications to enumerate the alternate data streams in a file or directory, these streams make excellent locations to hide malicious data. 
This vulnerability becomes very severe when compounded by the problem of exclusive locking.

Modern AV and EDR will still scan ADS if present, but ADS may still present a challenge to users or investigators.

 ---

### Hashing

Hashing applications may not consider ADS when computing a hash of a file leading to a potential false trust.
Some file integrity programs do not take the alternate data stream into account when making their calculations. 
This means that while they do protect the integrity of the contents of the primary, unnamed stream, they do not protect the integrity of alternate data streams.

---

### Exclusive Locks

An exclusive lock is a way for a process running on Windows to exclude all other processes from reading or writing to a particular file.
Each stream in a file has separate lock attributes, but very few Windows applications distinguish between the locks on unnamed and named streams. 
So, a program unaware of alternate streams and locks only looks at the primary stream will incorrectly believe it cannot access the file. 

- A user could find a file with that has been exclusively locked by a system process that they have permission to write to.
- They can then use a stream-aware application like Notepad to add data to a named stream. For extra security, the user will give the stream a very complicated name.
- Alternate data stream scanners will not be able to find the named stream on the locked file. Furthermore, because the primary stream is locked, the file cannot be deleted, moved, or renamed as long as the process is running. 
- The user will be able to read and write data from the alternate stream using the complicated name that he or she selected, even while the primary stream remains locked, evading detection of scanners.

---

### Denial of Service

Alternate streams also provide a very effective means to conduct a denial-of-service attack on a host. 
Because most Windows applications cannot show the size of any alternate streams on a file, a malicious user could easily hide an extremely large file in a named stream so that another user could not detect its presence.
The severity of the problem can escalate significantly for a stream placed on a file exclusively locked by the system, because many popular scanning applications cannot find a named stream on an exclusively locked file.

---

## Default Stream

The _default_ or _unnamed_ stream is always referenced with "$DATA" but additional streams can be created. 

ex.

```powershell
filename.txt:$DATA
```
```powersehll
filename.txt:stream
```

## Common Stream Example

### Zone.Identifier

A common use of ADS in Windows is to specify the "[Zone.Identifier](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-fscc/6e3f7352-d11c-4d76-8c39-2516a9df36e8)" stream. This stream stores Microsoft's "Mark of the Web" which if present forces the user to manually unblock the file before it can execute.

The data in the Zone.Identifier stream should looks like this:

```
[ZoneTransfer]

ZoneId=3
ReferrerUrl=
HostUrl=
```

More info about [Security Zones](https://learn.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/platform-apis/ms537183(v=vs.85)?redirectedfrom=MSDN)

![Zone](https://github.com/aalex954/ADS-Example/assets/6628565/51ef234f-8f50-4394-a3a1-5333ea1eedc0)


---

## Identifying Streams

### See the streams for all files in the current directory

```powershell
Get-Item -Path .\* -stream *
```

or 

```powershell
gi -path .\* -stream *
```

or even better

```powershell
Get-ChildItem -recurse | ForEach { Get-Item $_.FullName -stream * } | Where stream -ne ':$DATA'
```

![1](https://github.com/aalex954/ADS-Example/assets/6628565/bd618353-8ccf-427e-877d-f4710372dbf2)

---

## Working with text streams

### Get and Set text to a new stream

Echo some text into a new text file

```powershell
echo "super NOT secret text" > ".\super_sneaky_hidden_image.txt"
```

![3](https://github.com/aalex954/ADS-Example/assets/6628565/a90d4938-382f-4a78-9458-e3b1a67e939b)

---

Create a new stream and set its text contents

```powershell
Set-Content "super_sneaky_hidden_image.txt" -Stream SECRET
```

Get the original data ("$DATA")

```powershell
Get-Content "super_sneaky_hidden_image.txt" -Stream "$Data"
```

Get the new stream ("SECRET")

```powershell
Get-Content "super_sneaky_hidden_image.txt" -Stream SECRET
```

![4](https://github.com/aalex954/ADS-Example/assets/6628565/bca55cab-244f-4528-9a67-306c4fc705b6)

---

## Executables and streams

Although ADS can be used to hide executable code in other file types modern EDR solutions will/should check for ADS and also scan those streams. 
Still, a benefit to using ADS to hide malicious code is that if the file is hashed it will still produce the expected result since the ADS is not used for hashing. 

### Set an executable to a new stream

Put calc.exe into a new stream called "a_calculator"

```powershell
Set-Content -path ".\super_sneaky_hidden_image.txt" -value $(Get-Content $(Get-Command calc.exe).Path -readcount 0 -encoding byte) -encoding byte -stream a_calculator
```

![5](https://github.com/aalex954/ADS-Example/assets/6628565/2971a4c1-82b9-40f2-8c6f-301db2431540)

### Call an executable from a specified stream

Create and call a process from the stream "a_calculator"

```powershell
wmic process call create $(Resolve-Path .\super_sneaky_hidden_image.txt:a_calculator)
```


![6](https://github.com/aalex954/ADS-Example/assets/6628565/8d78bea2-302d-4824-98cc-272856d913c9)


