---
layout: page
title: "WinSuperMem"
subheadline: "New Tool Tuesday #1"
teaser: "The fastest Windows memory forensic artifact collector."
categories:
tags: 
header:
    title: "New Tool Tuesday #1"
    background-color: "#EFC94C;"
    #pattern: pattern_concrete.jpg
    image_fullwidth: /new-tool-tuesday/winSuperMem/header_cyber-sec.jpg
    caption: Image from unsplash
    caption_url: https://unsplash.com/photos/XJXWbfSo2f0
---

[WinSuperMem](https://github.com/CrowdStrike/SuperMem).. aka SuperMem.. Is a python tool developed by James Lovato at CrowdStrike to analyze Windows memory images. What makes this tool unique is the collection of forensic tools running under the hood. CrowStrike has glued together several existing forensic tools to produce the most efficient forensic tool at finding and extracting forensic artifacts from only Windows memory images!

CrowdStrike has glued together the following 7 forensic tools:

1. Volatility 2
2. Volatility 3
3. Bulk Extractor
4. Strings
5. EVTXtract
6. Plaso
7. Yara

All of the tools listed above get utilized in the WinSuperMem.py python script and are executed against a Windows memory image. You might be thinking, &quot;Wow 7 tools.. That will take a long time to complete an analysis&quot; and you would be wrong. This tool takes advantage of python&#39;s threading module to set a default thread count of 12, making this tool speedy!

## Tool Setup:

I won&#39;t be giving step by step instructions, rather share my experience in setting up this tool. As a side note, I set this tool up on my Kali linux machine. The github page for [SuperMem](https://github.com/CrowdStrike/SuperMem#installation) made things easy as all of the tools required are listed under the [Installation section](https://github.com/CrowdStrike/SuperMem#installation). You will need to have both python 2 and 3 installed to use Volatility 2 and 3.

Below are links to the tool requirements of where I got them:

- Volatility 3 - [https://github.com/volatilityfoundation/volatility3](https://github.com/volatilityfoundation/volatility3)
- Volatility 2 - [https://github.com/volatilityfoundation/volatility](https://github.com/volatilityfoundation/volatility)
- Volatility 2 Community Plugins - included with above download
  - located in &quot;/volatility/volatility/plugins/&quot;
- Bulk Extractor - preinstalled by Kali
- Plaso - [https://github.com/log2timeline/plaso/tree/main/tools](https://github.com/log2timeline/plaso/tree/main/tools)
  - You will need log2timeline.py and psort.py
- Yara - apt install
- Strings - preinstalled by Kali
- EVTxtract - [https://github.com/williballenthin/EVTXtract](https://github.com/williballenthin/EVTXtract)

After installing all of the requirements, open up file winSuperMem.py in a code editor and modify the global file paths starting on line 23 and ending on line 33.

Below is my global file paths:

```
# Globals Likely Needing Updated
 THREADCOUNT = 12
 EVTXTRACTPATH = &quot;/usr/local/bin/evtxtract&quot;# installed with pip
 VOL3PATH = &quot;/home/adam/downloaded\_Tools/volatility3/vol.py&quot;# changed
 VOL2PATH = &quot;/home/adam/downloaded\_Tools/volatility/vol.py&quot;# changed
 VOL2EXTRAPLUGINS = &quot;/home/adam/downloaded\_Tools/volatility/volatility/plugins/&quot;# changed
 BULKPATH = &quot;/usr/bin/bulk\_extractor&quot;
 LOG2TIMELINEPATH = &quot;/home/adam/downloaded\_Tools/plaso-20210606/tools/log2timeline.py&quot;# changed; plaso
 PSORTPATH = &quot;/home/adam/downloaded\_Tools/plaso-20210606/tools/psort.py&quot;# changed; plaso
 YARAPATH = &quot;/usr/bin/yara&quot;
 STRINGSPATH = &quot;/bin/strings&quot;
 YARARULESFILE = &quot;/path/to/yara/Yarafile.txt&quot;# i don&#39;t have my own yara rules file.. sad
```

The global file paths tell WinSuperMem where our tools are located on our machine. With all of the file paths configured, we are ready to run the tool.

## Tool Execution:

WinSuperMem has 3 different modes of execution. You can either do a Quick Triage, Full Triage, or a Comprehensive Triage. The difference in each triage is the number of forensic tools and volatility plugins that will be utilized.

### Quick Triage:

A Quick Triage will only use tools: volatility3, bulk extractor, and strings.

The following Volatility3 plugins will be used:

- pstree.PsTree
- cmdline.CmdLine
- callbacks.Callbacks
- svcscan.SvcScan
- registry.userassist.UserAssist
- envars.Envars
- handles.Handles
- modules.Modules
- dlllist.DllList
- getsids.GetSIDs
- getservicesids.GetServiceSIDs
- malfind.Malfind
- pslist.PsList
- registry.hivelist.HiveList
- ssdt.SSDT
- registry.hivescan.HiveScan

### Full Triage:

In addition to the tools/plugins used in a Quick Tirage, a Full Triage will use tools: Volatility2, EVTxtract, full registry and file dump with Volatility3, collect network IOCs, Plaso.

Additional Volatility3 plugins used:

- windows.modscan.ModScan
- windows.mutantscan.MutantScan 
- windows.psscan.PsScan
- windows.driverscan.DriverScan
- windows.symlinkscan.SymlinkScan 
- windows.driverirp.DriverIrp 
- windows.netscan.NetScan
- windows.filescan.FileScan
- windows.poolscanner.PoolScanner


Volatiltiy2 plugins used:

- amcache 
- getsids 
- clipboard 
- cmdscan
- consoles 
- ldrmodules 
- mftparser 
- psxview 
- shellbags 
- shutdowntime 
- indx 
- logfile 
- prefetchparser 
- schtasks
- sessions 
- shimcachemem 
- shimcache
- sockets
- sockscan 
- threads 
- usnjrnl 
- autoruns 
- connections 
- connscan
- hollowfind 
- malthfind 
- timeliner 
- apihooks 
- messagehooks

### Comprehensive Triage:

In addition to the tools/plugins used in a Full Triage, a Comprehensive Triage will use tools: 

- Will dump DLLs, Processes, and Modules with Volatility3 
- Yara

## Tool Efficiency:

WinSuperMem by default displays verbose output with a loading bar, so you always know where WinSperMem is in the execution process and what tool is currently executing. The script execution time is also calculated and displayed at the end. The best part of this tool is it&#39;s logging, all of the verbose output and commands executed are recorded in the &#39;Logging.log&#39; file by default.

![winSupermem_load_bar](/images/new-tool-tuesday/winSuperMem/winSupermem_load_bar.png)

The command to start running this tool is very simple:

`python3 winSuperMem.py -f evidence.mem -o output/ -tt 3`

- -f specifies the memory image file
- -o specifies the directory name to output the forensic artifacts
- -tt specifies the execution mode, 3 being comprehensive.

I tested the speed of WinSuperMem by running it against a 2GB memory dump file using comprehensive move. WinSuperMem completed execution in 11 minutes!

## Tool Challenge:

In order to test the ability of this tool, I challenged myself to complete a TryHackMe room called [Forensics](https://tryhackme.com/room/forensics). This room has a &#39;medium&#39; difficulty and involves analyzing a memory dump and collecting IOCs.

After downloading the memory dump, I executed the following command:

`python3 winSuperMem.py -f victim.raw -o output/ -tt 3`

From the screenshot below, I was able to get a lot of artifacts within 9 minutes: 

![dir_list](/images/new-tool-tuesday/winSuperMem/dir_list.png)

I was able to complete the entire room by only using artifacts from Volatility2, Volatility3, and Strings.

Volatility3 artifacts files used:

- windows.info.csv
- windows.PsList.csv
- windows.netScan.csv
- windows.malFind.csv
- windows.envars.csv

Volatility2 artifacts files used:

- Shellbags.out

All of the IOCs were found within the strings.ascii file.

## Conclusion:

WinSuperMem is an excellent forensic tool. You will save an enormous amount of time by having this tool automate all of the typical commands you would run with Volatility alone. I plan to use WinSuperMem anytime I am faced with a memory image file as a starting point for analysis.