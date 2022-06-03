---
title: '[Notes] Volatility Cheatsheet'
date: 2022-05-02 8:01 PM
categories: [Notes, Forensics]
tags: [forensics, volatility, tools, cheatsheet, linux]
permalink: /:categories/:title
---

Below are some of the more commonly used plugins from Volatility 2 and their Volatility 3 counterparts.

### OS Information

#### imageinfo

- Volatility 2

```
vol.py -f “/path/to/file” imageinfo
vol.py -f “/path/to/file” kdbgscan
```

- Volatility 3

`vol.py -f “/path/to/file” windows.info`

Output differences:  

- Volatility 2: Additional information can be gathered with kdbgscan if an appropriate profile wasn’t found with imageinfo  
- Volatility 3: Includes x32/x64 determination, major and minor OS versions, and kdbg information

### Process Information

#### pslist

- Volatility 2

```
vol.py -f “/path/to/file” ‑‑profile <profile> pslist
vol.py -f “/path/to/file” ‑‑profile <profile> psscan
vol.py -f “/path/to/file” ‑‑profile <profile> pstree
vol.py -f “/path/to/file” ‑‑profile <profile> psxview
```

- Volatility 3

```
vol.py -f “/path/to/file” windows.pslist
vol.py -f “/path/to/file” windows.psscan
vol.py -f “/path/to/file” windows.pstree
```

Output differences:  

- Volatility 2: Additional process lists with psxview  
- Volatility 3: Does not include a direct psxview equivalent

#### procdump

- Volatility 2

`vol.py -f “/path/to/file” ‑‑profile <profile> procdump -p <PID> ‑‑dump-dir=“/path/to/dir”`

- Volatility 3

`vol.py -f “/path/to/file” -o “/path/to/dir” windows.dumpfiles ‑‑pid <PID>`

Output differences:  

- Volatility 2: Just outputs specified PID 
- Volatility 3: Dumps exe and associated DLLs

#### memdump

- Volatility 2

`vol.py -f “/path/to/file” ‑‑profile <profile> memdump -p <PID> ‑‑dump-dir=“/path/to/dir”`

- Volatility 3

`vol.py -f “/path/to/file” -o “/path/to/dir” windows.memmap ‑‑dump ‑‑pid <PID>`

#### handles

- Volatility 2

`vol.py -f “/path/to/file” ‑‑profile <profile> handles -p <PID>`

- Volatility 3

`vol.py -f “/path/to/file” windows.handles ‑‑pid <PID>`

Output differences:  

- Volatility 2: Offset(V), PID, handle, access, type, details  
- Volatility 3: PID, process, offset, handlevalue, type, grantedaccess, name

#### dlls

- Volatility 2

`vol.py -f “/path/to/file” ‑‑profile <profile> dlllist -p <PID>`

- Volatility 3

`vol.py -f “/path/to/file” windows.dlllist ‑‑pid <PID>`

Output differences:  

- Volatility 2: PID, command line, base, size, loadcount, loadtime, path  
- Volatility 3: PID, process, base, size, name, path, loadtime, file output

#### cmdline

- Volatility 2

```
vol.py -f “/path/to/file” ‑‑profile <profile> cmdline
vol.py -f “/path/to/file” ‑‑profile <profile> cmdscan
vol.py -f “/path/to/file” ‑‑profile <profile> consoles
```

- Volatility 3

`vol.py -f “/path/to/file” windows.cmdline`

Output differences:  

- Volatility 2: process name, PID, commandline; cmdscan includes 
  application, flags, process handle; consoles contains C:\ listing, 
  original titles, screen position and command history information  
- Volatility 3: PID, process name, args

### Network Information

#### netscan

- Volatility 2

```
vol.py -f “/path/to/file” ‑‑profile <profile> netscan
vol.py -f “/path/to/file” ‑‑profile <profile> netstat
```

##### - _XP/2003 SPECIFIC_

```
vol.py -f “/path/to/file” ‑‑profile <profile> connscan
vol.py -f “/path/to/file” ‑‑profile <profile> connections
vol.py -f “/path/to/file” ‑‑profile <profile> sockscan
vol.py -f “/path/to/file” ‑‑profile <profile> sockets
```

- Volatility 3

```
vol.py -f “/path/to/file” windows.netscan
vol.py -f “/path/to/file” windows.netstat
```

Note: The XP/2003 specific plugins are deprecated and therefore not available in Volatility 3

### Registry

#### hivelist

- Volatility 2

```
vol.py -f “/path/to/file” ‑‑profile <profile> hivescan
vol.py -f “/path/to/file” ‑‑profile <profile> hivelist
```

- Volatility 3

```
vol.py -f “/path/to/file” windows.registry.hivescan
vol.py -f “/path/to/file” windows.registry.hivelist
```

#### printkey

- Volatility 2

```
vol.py -f “/path/to/file” ‑‑profile <profile> printkey
vol.py -f “/path/to/file” ‑‑profile <profile> printkey -K “Software\Microsoft\Windows\CurrentVersion”
```

- Volatility 3

```
vol.py -f “/path/to/file” windows.registry.printkey
vol.py -f “/path/to/file” windows.registry.printkey ‑‑key “Software\Microsoft\Windows\CurrentVersion”
```

#### hivedump

- Volatility 2

`vol.py -f “/path/to/file” ‑‑profile hivedump -o <offset>`

- Volatility 3

I’m not sure if this capability exists in Vol3; however, you 
may be able to extract registry hives using filedump with the offset

### Files

#### filescan

- Volatility 2

`vol.py -f “/path/to/file” ‑‑profile <profile> filescan`

- Volatility 3

`vol.py -f “/path/to/file” windows.filescan`

#### filedump

- Volatility 2

```
vol.py -f “/path/to/file” ‑‑profile <profile> dumpfiles ‑‑dump-dir=“/path/to/dir”
vol.py -f “/path/to/file” ‑‑profile <profile> dumpfiles ‑‑dump-dir=“/path/to/dir” -Q <offset>
vol.py -f “/path/to/file” ‑‑profile <profile> dumpfiles ‑‑dump-dir=“/path/to/dir” -p <PID>
```

- Volatility 3

```
vol.py -f “/path/to/file” -o “/path/to/dir” windows.dumpfiles
vol.py -f “/path/to/file” -o “/path/to/dir” windows.dumpfiles ‑‑virtaddr <offset>
vol.py -f “/path/to/file” -o “/path/to/dir” windows.dumpfiles ‑‑physaddr <offset>
```

### Miscellaneous

#### malfind

- Volatility 2

`vol.py -f “/path/to/file” ‑‑profile <profile> malfind`

- Volatility 3

`vol.py -f “/path/to/file” windows.malfind`

Output differences:  

- Volatility 2: PID, process name, address, VAD tags, hexdump, and shellcode  
- Volatility 3: PID, process name, process start, protection, commit charge, privatememory, file output, hexdump disassembly

#### yarascan

- Volatility 2

`vol.py -f “/path/to/file” yarascan -y “/path/to/file.yar”`

- Volatility 3

```
vol.py -f “/path/to/file” windows.vadyarascan ‑‑yara-rules <string>
vol.py -f “/path/to/file” windows.vadyarascan ‑‑yara-file “/path/to/file.yar”
vol.py -f “/path/to/file” yarascan.yarascan ‑‑yara-file “/path/to/file.yar”
```

---

### Sources

1. [ONFVP Blog](https://blog.onfvp.com/post/volatility-cheatsheet/)
