# ADS-Example
NTFS Alternate Data Streams Example

![yo_dawg](https://github.com/aalex954/ADS-Example/assets/6628565/47a7a6e5-cb3c-42fc-972e-bfe12e6bed58)


Alternative NTFS streams or Alternitive Data Streams are used to store additional information (meta data) about a file. 

The _default_ or _unnamed_ stream is always referenced with "$DATA" but additional streams can be created. 

ex.

```powershell
filename.txt:$DATA
```
or
```powersehll
filename.txt:stream
```
## Zone.Identifier

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

## Working with text streams

### Get and Set text to a new stream

Echo some text into a new text file

```powershell
echo "super NOT secret text" > ".\super_sneaky_hidden_image.txt"
```

![3](https://github.com/aalex954/ADS-Example/assets/6628565/a90d4938-382f-4a78-9458-e3b1a67e939b)

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


