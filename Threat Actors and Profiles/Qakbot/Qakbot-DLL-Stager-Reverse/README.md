Reverse engineer of Qakbot DLL Stager
===============================================


<p align="center" width="100%">
    <img width="100%"src="https://s3.ca-central-1.amazonaws.com/esentire-dot-com-assets/assetsV3/Blog/Blog-Images/QakbotCampaign_Picture3.png"> 
</p>

Initialization
--------------

This first stage starts checking for ***DLL\_PROCESS\_ATTACH***.

If this is the case, it will proceed to do its first initialization
procedures, which include:

1.  Create a heap object which will be used later for allocations of
    dynamic structs or strings.

2.  Initialize a character set global string, which will be used for
    string manipulation.

3.  Copy the Import directory table from the DLL module, to a new global
    buffer (array of **IMAGE\_IMPORT\_DESCRIPTOR** references through
    DataDirectory in the Optional Header).

4.  Resolve with API hashing a dynamic kernel32.dll function table, with
    a different number of APIs used during the execution flow.

5.  Check with GetFileAttributes, for ***"C:\\INTERNAL\\\_\_empty".***
    This has been described as a way to check for Windows Defender
    emulation.

After all these steps have been done successfully, Qakbot will use the
newly created kernel32 IAT to call CreateThread with the ***main
malicious thread.***

It is important to clarify that Qakbot also uses string decryption for
both WCHAR and CHAR, which use two wrappers each.

As of May/June 2022, one of the new functions that has been added is the
check for an environment variable SELF\_TEST1. This serves as a flag for
doing specific checks in terms of the path of the module itself, which
seems to be mainly for debugging purposes, since it pops up a MessageBox
with *"Self Check", "Self Check ok!"* and *"ERROR: GetModuleFileNameW()
failed with error: 0".*

In this sense, the debug string describes the core functionality, which
is a custom GetModuleFilenameW() implemented using CommandLineToArgvW.

![](.//media/image2.jpg)

> ![](.//media/image3.jpg)
> ![](.//media/image4.png)

![](.//media/image5.jpg)

The main malicious thread (First stage): 
-----------------------------------------

After successful checks, the main malicious thread will call the same
API hashing function that rebuilds a new IAT for more modules including:
ntdll.dll, user32.dll, netapi32.dll, advapi32.dll, shlwapi.dll,
userenv.dll, and ws2\_32.dll

*(This last module ws2\_32.dll has been recently added in February/March
2022, to replace inet\_ntoa which was in the imports of the DLL so you
could just use xrefs to easily reverse the structure that stored both
the C2 IPs and ports, more of that at the end of the paper, in
additional details.)*

Then it will proceed to build a structure based on the information from
the computer, where all types of information are initialized in the
structure members for different purposes, like random names for mutexes
or events.

Some of the information includes: Computer Name, Volume information,
Primary Domain Controller Name, Privileges, Module name, Type of Process
(WOW64 or x64 binary), TickCount, Os Version Info and much more.

Additionally, it sets up environment variables as needed for
USERPROFILE, TEMP and SystemDrive.

> ![](.//media/image6.jpg)
>
> struct merserne\_twister{ uint32\_t twister\[624\]; uint32\_t index;
>
> };
```
+-----------------------------------------------------------------------+
| struct initialization                                                 |
|                                                                       |
| {                                                                     |
|                                                                       |
| > OSVERSIONINFOEXA info\_struct\_os;                                  |
| >                                                                     |
| > DWORD m\_procesorArchitecture;                                      |
| >                                                                     |
| > DWORD m\_spawn\_regsvr32\_instance\_check; //Used for certain       |
| > checks in operations.                                               |
| >                                                                     |
| > DWORD m\_detected\_flag; //Used for certain checks in operations.   |
| >                                                                     |
| > DWORD m\_is\_wow64\_process;                                        |
| >                                                                     |
| > DWORD m\_crc32\_hash\_computer\_info;                               |
| >                                                                     |
| > CHAR m\_result\_random\_computer\_info\[32\]; //Computer            |
| > information related                                                 |
| >                                                                     |
| > WCHAR m\_result\_random\_computer\_info\_wchar\[32\]; //Computer    |
| > information related wchar                                           |
| >                                                                     |
| > TOKEN\_USER \*m\_current\_process\_token;                           |
| >                                                                     |
| > WCHAR m\_account\_name\[128\]; //Account name for current option    |
| >                                                                     |
| > DWORD m\_checks\_sid\_option; //Depends on SID, used for different  |
| > operations.                                                         |
| >                                                                     |
| > LPWSTR m\_name\_netbios\_buffer;                                    |
| >                                                                     |
| > DWORD m\_dc\_name\_ptr; //Domain controller related                 |
| >                                                                     |
| > DWORD m\_buffer\_type\_netDC; //Domain controller related           |
| >                                                                     |
| > DWORD m\_dll\_module\_handle;//Handle                               |
| >                                                                     |
| > WCHAR m\_current\_container\_executable\_full\_path\[262\];         |
| > //Executable that loaded dll                                        |
| >                                                                     |
| > PWCHAR m\_ptr\_current\_folder\_container; //ptr to folder of       |
| > container executable.                                               |
| >                                                                     |
| > WCHAR m\_current\_folder\_container\_executable\[262\];             |
| > //Executable that loaded dll folder                                 |
| >                                                                     |
| > DWORD m\_unknown; //Only referenced once, usage not clear.          |
| >                                                                     |
| > merserne\_twister mersenne\_tick\_count\_info; //Based on tick      |
| > count info.                                                         |
| >                                                                     |
| > CHAR m\_pos\_bot\_id\[16\]; //Used in C2 comms functions, possible  |
| > bot info.                                                           |
| >                                                                     |
| > DWORD m\_sid\_subauthority\_option;                                 |
| >                                                                     |
| > WCHAR m\_windows\_dir\_path\[261\]; //Windows Dir path              |
| >                                                                     |
| > WCHAR m\_temp\_path\[261\]; //Temp path                             |
| >                                                                     |
| > WCHAR m\_user\_profile\_path\[262\]; //User profile                 |
| >                                                                     |
| > UINT m\_current\_proc\_ID;                                          |
| >                                                                     |
| > WCHAR m\_current\_container\_executable\_wchr\[262\]; //Container   |
| > exec wchr version                                                   |
| >                                                                     |
| > DWORD m\_check\_system\_metrics;                                    |
| >                                                                     |
| > PWCHAR m\_current\_folder\_wchar\_ptr; //Container executable       |
| > folder wchar ptr                                                    |
| >                                                                     |
| > CHAR m\_seed\_mersenne\_twister\_string\_32\[32\];//Random 1        |
| >                                                                     |
| > CHAR m\_second\_seed\_random\[32\]; //Random 2                      |
| >                                                                     |
| > DWORD m\_ored\_total\_process\_detection; //Bitmask for first       |
| > process detection.                                                  |
| >                                                                     |
| > BYTE unused\_bytes\[256\]; //Not initialized in constructor,        |
| > padding bytes.                                                      |
| >                                                                     |
| > WCHAR m\_ComputerName\[148\]; //Current computer name.              |
|                                                                       |
| };                                                                    |
+-----------------------------------------------------------------------+
```
In the final part from this structure constructor, there will be a check
for certain processes as a detection mechanism, mainly done through an
array of structures where each element will store a value that will
decrypt the process names strings separated by **';'** character, using
the WCHAR version of this function.

For example: **"explorer.exe;notepad.exe;cmd.exe",** each process names
separated by ';' one of them will be stored as a char\* inside the
ptrToProcessNames char\*\* (char\*\[ \])

```
+---------------------------------------------------------------------+
| struct ProcessDetection //Per processes detection structure.        |
|                                                                     |
| {                                                                   |
|                                                                     |
| > DWORD dwBitmask; //Total Bitmask                                  |
| >                                                                   |
| > DWORD dwEncryptedNameIndex; //can store several names             |
| >                                                                   |
| > DWORD dwCountProcessNames; //Number of processes separated by ';' |
| >                                                                   |
| > CHAR\*\* ptrToProcessNames; //Process names separated by ';'      |
|                                                                     |
| };                                                                  |
|                                                                     |
| struct ContainerProcessDetection                                    |
|                                                                     |
| {                                                                   |
|                                                                     |
| > DWORD dwBitmaskDetection;                                         |
| >                                                                   |
| > DWORD dwNumDetectionStructures;                                   |
|                                                                     |
| ProcessDetection \*arrDetectStructs; };                             |
+---------------------------------------------------------------------+
```

The value in ***dwBitMaskDetection*** inside the
***ContainerProcessDetection*** struct will serve as a bitmask, where
each time one of the processes is detected, it will be ORd in this
member with the current ProcessDetection member ***dwBitmask.***

All the processes detected this way include:

***ccSvcHst.exe, avgcsrvx.exe;avgsvcx.exe;avgcsrva.exe, MsMpEng.exe
,mcshield.exe, avp.exe;kavtray.exe
,egui.exe;ekrn.exe,bdagent.exe;vsserv.exe;vsservppl.exe***

***AvastSvc.exe,coreServiceShell.exe;PccNTMon.exe;NTRTScan.exe,SAVAdminService.exe;SavS
ervice.exe, fshoster32.exe,WRSA.exe,vkise.exe;isesrv.exe;cmdagent.exe***

***ByteFence.exe,MBAMService.exe;mbamgui.exe,fmon.exe,dwengine.exe;dwarkdaemon.exe;dw
watcher.exe***

![](.//media/image7.jpg)

This will be stored in the initialization struct, where it is labeled as
**m\_ored\_total\_process\_detection.** Each bitfield inside this
bitmask will be checked in different functions for specific operations
as it will be seen below.

Once everything above has been done successfully, certain checks are
done which determine the different variations for executing the "second
stager"function:

***- Code injection through entrypoint function hooking in newly created
suspended process:***

***-Service creation and register of malicious status handlers. -Direct
call to the second stager function.***

Let\'s inspect some of these methods, and how they are being used.

### Entrypoint function hooking and service status handling for code execution:

Qakbot will decide based on the bitfields checks found, which processes
to create for the code injection of the next important function.

In this sense, all the decrypted relative paths will use
ExpandEnvironmentStringsW to get a full path to the binary to be
executed, where 3 of them are returned through a WCHAR\*\* pointer
(WCHAR\*\[ \]), depending on the processes detected during the first
process detection phase.

Some decrypted paths include:
```
> ***\'%SystemRoot%\\SysWOW64\\explorer.exe\'***
>
> ***\'%SystemRoot%\\SysWOW64\\mobsync.exe\'***
>
> ***\'%SystemRoot%\\System32\\mobsync.exe\'***
>
> ***\'%SystemRoot%\\explorer.exe\'***
>
> ***\'%ProgramFiles%\\Internet Explorer\\iexplore.exe\'***
>
> ***\'%SystemRoot%\\SysWOW64\\OneDriveSetup.exe\'***
>
> ***\'%SystemRoot%\\SysWOW64\\msra.exe\'***
>
> ***\'%ProgramFiles(x86)%\\Internet Explorer\\iexplore.exe\'***
>
> ***\'%SystemRoot%\\System32\\OneDriveSetup.exe\'***
>
> ***\'%SystemRoot%\\System32\\msra.exe\'***
```

Once the process has been created with CreateProcess, using the
**CREATE\_SUSPENDED** flag, it uses NtCreateSection to create a section
with the size of the current DLL Optional Header SizeOfImage, which is
the size of the PE when it is mapped in memory.

In this sense, NtMapViewOfSection is used with this section handle to
map the new section in the current process address space and in the
remote address space.

After these steps finish successfully, VirtualAllocEx and
WriteProcessMemory are being used to allocate virtual memory for a newly
copied ***initialization*** struct inside our target process created,
changing its member ***m\_dll\_module\_handle*** to the address of the
mapped section in the external process.

Additionally, to deal with relocations, first the dll is copied to the
view of the section mapped in the current process using SizeOfImage in
the Optional Header.

and after it, proper relocations will be done for the DLL using both the
addresses of both views, effectively fixing the DLL for usage in the
external process.

After this process has finished successfully, Qakbot will proceed to
hook the entrypoint function, using the ThreadContext structure,
specifically the EAX register, which contains the entrypoint address of
the external process, patching the bytes with NtProtectVirtualMemory and
NtWriteVirtualMemory, and eventually resuming the main thread with the
hooked entrypoint in the remote process (the hooking function is inside
the mapped dll)

On the other hand, if the dll has system privileges, it will proceed to
start a service control dispatcher with StartServiceCtrlDispatcherA. In
this sense, this dispatcher will call RegisterServiceCtrlHandlerA and
set the current state of the service.

The most interesting detail is that, once the **SERVICE\_RUNNING**
status is set**,** the execution continues to the next important
function.

![](.//media/image8.png)

All of these executions lead to the ***Second Stager function*** itself, which
contains a lot of functionality that we will be describing next.

![](.//media/image9.png)
> ![](.//media/image10.jpg)

Here you can see in the screenshots above how precisely it is done the
operations for the mappings of the section which contains the current
stager dll.

To be completely precise, the HookFunction (shown below) will be
executed in the context of the remote mapped dll thanks to the
calculations shown in the BytesHook\[1\] operation. (shown above)

> ![](.//media/image11.png)

Additionally, one of the most important things to remember is the
***m\_detected\_flag*** member in the ***initialization*** struct.

This will be important for operations as you will see later, where
***both ServiceProc and the normal call to this function will set this
member to be 1, meanwhile, for the hook case, the value has to be 2.***

It is important to mention that the event signaled in the
***HookFunction*** will be checked while spawning and hooking processes,
and if the event is found, it will stop to try spawning new target
processes with the 3 attempts. Additionally the HookFunction will
resolve the IAT for this mapped dll, so all necessary functionalities
can be executed properly.

Inside the **Execution::FirstStager** function image, this function is
labeled as

**Injection::ManualMapDllAndHookEntrypoint()**

The second stage function: execution states and configurations.
---------------------------------------------------------------

![](.//media/image15.jpg)


Let\'s understand some of the most important functionality that can be seen in this procedure.

### The execution states of a Qakbot stager instance:

It is not too clear how it is possible to differentiate what would be
executed depending on certain conditions for a Qakbot instance.

For this purpose, in the second stage function, the Dll will check what
is going to continue to be executed, based on the stored config in the
current system, which involves events, mutexes and registry values. It
is also important to describe that before checking this

The execution state is set mainly with following 4 options:

1.  First instance (no container executable found in registry)

2.  Same container executable name as registry.

3.  Mutex based on computer info already exists (initialized in the
    execution state 3)

4.  Event already signaled (mainly done inside an exception handler
    function that we will see later in section)

All these conditions determine which parts of the code are going to be
executed, for now, we will only focus on the execution state 0, since
this is the one that relates to the first execution flow.

Additionally, I consider important to describe that before the execution
state is set, if the member in the initialization struct,
**m\_detected\_flag,** is not equal to 1 ***(Hook entrypoint case)***,
it will proceed to generate a new buffer in memory of the current
container executable, which will be used for later operations.

> ![](.//media/image16.jpg)
>
> ![](.//media/image17.jpg)

For each case, the results (assuming everything works out as intended)
are:

***- Execution state 0:*** Proceed to spreader component, then check for
member **m\_detected\_flag** != 1 ***(Hook Entrypoint case)***, if this
is the case, then it proceeds to delete schtasks/run key persistence set
in one of the generated regsvr32 execution instances, then proceed to
execute *the **Execution::ThirdStager** function, additionally
registering an exception handler**.***

***- Execution state 1:*** Check for member **m\_detected\_flag** != 1
***(Hook Entrypoint case)*** , if this is the case, then it proceeds to
delete schtasks/run key persistence from previously generated regsvr32
execution instances. After this proceeds to ***Execution::ThirdStager()
***

*- **Execution state 2:** Returns 0 (EXIT\_SUCCESS).*

***- Execution state 3:*** Proceeds to signal event, where the GUID uses
the computer info hash as seed.

Then proceeds to create the mutex that is relevant to achieve execution
state 2.

Replaces container executable if the value stored in the registry config
is not the same, storing the current one and then deleting the file from
the previous value.

After that, it will execute ***Execution::ThirdStager***, *additionally
registering an exception.*

You can see all these details graphically, in the image at the start of
this section, showing the ***Execution::SecondStager() function.***

### Configuration storage in registry for preservation between instances:

Qakbot at this point initializes a structure that will be used for
storing and retrieving important information from the Windows Registry.
This is very important to remember because it is used for retrieving
specific configuration that can be used by different instances of this
stager.

+-----------------------------------------------------------------------+
| struct ContainerSubkeyToStore                                         |
|                                                                       |
| {                                                                     |
|                                                                       |
| > DWORD dwLengthSubkeyValue;                                          |
| >                                                                     |
| > BYTE EncryptedSubkeyValue\[260\];                                   |
| >                                                                     |
| > BYTE HashKeyToDecryptSubkey\[4\]; //This is based on CRC32 hash.    |
| >                                                                     |
| > HKEY hKeyUsers;                                                     |
|                                                                       |
| };                                                                    |
|                                                                       |
| //Size is allocated dynamically for each case, struct mainly for      |
| static analysis. struct \_\_unaligned EncryptedConfig                 |
|                                                                       |
| {                                                                     |
|                                                                       |
| > BYTE typePayload; //Takes several values: 5, 2, 3, 4.               |
| >                                                                     |
| > BYTE bValidPayload; //Verified in the found handlers to do          |
| > decryption.                                                         |
| >                                                                     |
| > DWORD dwFullSizePayload; //Size of the payload.                     |
| >                                                                     |
| > //IMPORTANT: The two last members change the size depending on each |
| > case.                                                               |
| >                                                                     |
| > BYTE PayloadBuffer\[1\]; //Actual buffer to the payload buffer.     |
|                                                                       |
| BYTE RandomBufferGen\[1\]; //Generated buffer inside                  |
| Random::GenerateRandomBuffer };                                       |
+-----------------------------------------------------------------------+

During the constructor phase, the member HashKeyToDecryptSubkey is used
to encrypt the subkeyValue to be created. The HashKeyToDecryptSubkey
value comes from a CRC32 based on the computer information.

This hash value is used for each BYTE in the encryption/decryption
process, working it through from i = 0, with i & 3, this way we enforce
always looping the 4 bytes of the hash.

When certain informations needs to be stored, the structure
EncryptedConfig is used, where an specific index is passed through the
function to create a SHA1 hash, using this hash as salt for generating
the encryption key, which is used for encrypting the entire
***EncryptConfig*** buffer before being stored with an RC4 encryption
algorithm.

After to then encrypt it and store it as a value in the corresponding
value, where

***EncryptedSubkeyValue, HashKeyToDecryptSubkey*** and
***dwLengthSubkeyValue*** are all used, from the
***ContainerSubkeyToStore***, where RegOpenKeyExA and RegSetValueExA are
used for this purpose.

> ![](.//media/image18.jpg)
>
> ![](.//media/image19.png)

Configuration storage in resources:
-----------------------------------

At this point, the structure that I labeled as ***ResourceDecryption***
below, will be initialized. Everybody knows where the config is usually
stored and how it usually is decrypted in the first instance, but what
is the layout of the structure that uses it?

Simple, it\'s the ***ExtractedElements*** structures shown below, which
basically store the index element and the string of the element.

If the decrypted config stores as an example: ***\"10 = obama165\"***,
***indexElement*** would be 10 (as an integer) and ***stringElement***
will contain obama165, for the current ***element*** struct.

```
+-----------------------------------------------------------------------+
| struct ResourceDecryption //Struct for storing the valuable config.   |
|                                                                       |
| {                                                                     |
|                                                                       |
| > DWORD dwCountExtracted;                                             |
| >                                                                     |
| > DWORD dwUnknown;                                                    |
| >                                                                     |
| > HMODULE hCurrentModuleDll;                                          |
| >                                                                     |
| > ExtractedElements \*ExtractedResourcesConfig; //config from         |
| > resources of stager.                                                |
|                                                                       |
| ExtractedElements \*ExtractedCfgFileConfig; //config from .cfg };     |
+=======================================================================+
| struct element                                                        |
|                                                                       |
| {                                                                     |
|                                                                       |
| > DWORD indexElement; char \*stringElement;                           |
|                                                                       |
| };                                                                    |
|                                                                       |
| struct ExtractedElements                                              |
|                                                                       |
| {                                                                     |
|                                                                       |
| > DWORD numElements; element \*Elements;                              |
|                                                                       |
| };                                                                    |
|                                                                       |
| struct ResourceBufferStruct //Struct used for in-memory decryption of |
| resource.                                                             |
|                                                                       |
| { wchar\_t m\_path\_file\[512\];                                      |
|                                                                       |
| > DWORD m\_sha1\_hash\_buffer\[8\];                                   |
|                                                                       |
| WORD dwSizeKey;                                                       |
|                                                                       |
| BYTE padding1\[2\];                                                   |
|                                                                       |
| > void\* m\_ptrToDynamicAllocBuffer;//In-memory buffer of the         |
| > resource                                                            |
| >                                                                     |
| > DWORD m\_dwSizeDecrypted; //In-memory size of the resource          |
| > decrypted                                                           |
| >                                                                     |
| > DWORD m\_dwSizeDecryptedCopy;                                       |
| >                                                                     |
| > HANDLE hHandle;                                                     |
|                                                                       |
| BYTE m\_dwOptionDecryption;                                           |
|                                                                       |
| BYTE padding2\[3\];                                                   |
|                                                                       |
| > DWORD m\_dwInitHeaderAplibCheck; //Check done before using APLIB    |
| > compression.                                                        |
| >                                                                     |
| > DWORD m\_dwNumberElements;                                          |
| >                                                                     |
| > DWORD m\_dwNumberOfCurrentElements;                                 |
| >                                                                     |
| > ExtractedElements \*\* BufferElements;                              |
|                                                                       |
| };                                                                    |
|                                                                       |
| //Important: This struct will be used again, e.g: injector.           |
+-----------------------------------------------------------------------+
```
I do not want to focus on all the possible methods that are used for
decrypting and decompressing the payloads depending on each specific
case (BriefLZ can be still used and size of 0x28 is also checked for the
resource buffer to do an entirely different decryption operation, RC4 is
also done in this case).

I want to focus on how the config is extracted for the main usual case,
which is what most people are interested in:

A key is decrypted at runtime with one of the CHAR decrypt string
wrapper, which is important for decryption.

This key (20 bytes) will be processed using SHA-1 algorithm, and then it
will decrypt the buffer with RC4 algorithm, and it will additionally use
SHA-1 for integrity checks, the core payload is finally obtained to be
parsed correctly for proper usage by the Qakbot stager, which is mainly
done through the ***ExtractedElements*** structure.

To reimplement it, it is just required for you to extract this key
statically (or dynamically), and then replicate the same as what it is
executed in the function. I will show below how it looks so you get a
feeling of it when you are reversing it.

> ![](.//media/image20.jpg)

Above you can see the function that is in charge of both decrypting and
checking the integrity of the decrypted payload, in my IDB this is
labeled as ***Decryption::DecryptBufferAndCheckIntegrity,** as you can
see* in the image)

I recommend the following resource for more description related to this
specific decryption part:
[[https://darkopcodes.wordpress.com/2020/06/07/malware-analysis-qakbot-part-2/]{.underline}](https://darkopcodes.wordpress.com/2020/06/07/malware-analysis-qakbot-part-2/)

This resource describes a little bit better how old the code is, and how
it has not really changed that much. However, it will most likely change
in the future, so it is important that you reverse it on your own, so
you can get a grasp of it

AV detection checks and methods to manipulate relevant stager files:
--------------------------------------------------------------------

Qakbot stager will try to detect aswhookx.dll and aswhooka.dll in the
current process where it is loaded, if a related process has been found
in the detection bitmask described before.

> ![](.//media/image21.jpg)

At this point I consider it important to describe the function

***Storage::GenerateNewDllandRegistryPaths***, because it is used in
other components such as the network spreader.

First of all, a random dll name will be generated using different
operations that use the current account name and CRC32 hashing. This
random dll name will be concatenated with a newly created folder name,
to eventually generate a working path for usage.

> ![](.//media/image22.jpg)

Having in mind the picture above, you can see that the folder name
depends on the random dll name generated previously. This is shown in
the function ***Random::GenerateRandom16LengthString***. This random
folder will be created in the same function with CreateDirectoryW, if it
does not exist.

Additionally, if certain processes are found with the bitmask member
inside the initialization struct and the privileges are enough, the
folder is added in the following HKLM registry keys :

> ***1.SOFTWARE\\Microsoft\\Microsoft Antimalware\\Exclusions\\Paths***
>
> ***2.SOFTWARE\\Microsoft\\Windows Defender\\Exclusions\\Paths***

After this steps have finished, a ***ContainerSubkeyToStore***
constructor is called, where some of the information initialized at this
function is used as arguments, mainly using the Profile Path (the
profile path is obtained with ProfileImagePath at the start of the
function), the full path of the random dll and the crc32 hash generated
from the computer info.

Using this structure, some configuration is stored in the registry,
which includes decrypted c2 config information in the resources for both
the *current stager and .cfg file (if found), random dll path, and
time.*

> ![](.//media/image23.jpg)

At this point, depending on the member ***m\_detected\_flag*** described
before, it can work out some of the relevant stager files in different
ways. This also depends on the current SID of the current user
(**m\_checks\_sid\_option**).

***There are two main ways this is done by Qakbot:***

The first one, which is mainly done for the ***hook entrypoint case***,
involves using some in-memory buffer of a file, to create and write to
the current random dll.

The additional details of how this is generated for each case are
detailed on a previous chapter of this paper, but essentially, for this
case, the buffer used will be the ***current container executable.***

It is important to mention that this function will be very important for
regsvr32 execution, because it can generate new execution instances
through persistence, which will be described before

> ![](.//media/image24.png)

The second one is much more complex, and involves invoking different
function pointers, which are called in pairs, until one of them
succeeds.

Each one passes the random dll path and the current executable path that
has loaded the dll I will describe individually all the methods, without
a particular order:

1.-Generating bat file with contents:

***wmic process call create \'expand \<container executable\> \<random
dll path\>\'***, then creating a process with the bat file.

After it, it proceeds to try to delete the bat file.

2.-Creates a memory buffer of the container executable, and then it
proceeds to write it to the target dll, effectively generating a new
copy.

3\. Using a vbs file which is executed through cscript.exe using
ShellExecuteW. The main that this contains is the following vb code:
```
+-----------------------------------------------------------------------+
| Set objWMIService =                                                   |
| GetObject(\"winmgmts:\"&\"{impersonationLevel=impersonate}!\\\\.\\%co |
| ot\\cimv2\")                                                          |
|                                                                       |
| Set colFiles = objWMIService.ExecQuery(\"Select \* From CIM\_DataFile |
| Where Name = \<***container executable***\>\'\")                      |
|                                                                       |
| For Each objFile in colFiles objFile.Copy(***\<random dll path\>***)  |
|                                                                       |
| > Next                                                                |
+-----------------------------------------------------------------------+
```
4.-Use CopyFileW to copy files from the container executable to target
dll.

> ![](.//media/image25.jpg)

It is important to point out that one array entry will be added in an
dynamic array of structures after all this process has finished
correctly, increasing a counter. We do not really care about the layout
of this struct because it is not explicitly used, but we will point out
the usage of the counter in the next section, which is labeled as
***indexCryptoStruct***.

Network/Local accounts spreader component with run key persistence, and schtasks persistence:
---------------------------------------------------------------------------------------------
```
+-----------------------------------------------------------------------+
| struct NetworkSpreader                                                |
|                                                                       |
| {                                                                     |
|                                                                       |
| DWORD sid\_option; //Copied from m\_checks\_sid\_option inside the    |
| initialization struct.                                                |
|                                                                       |
| > ResourceDecryption \*structResources; //Overall config so far       |
| > extracted                                                           |
| >                                                                     |
| > PWTS\_SESSION\_INFOW pSessionInfo; //Extracted used inside          |
| > WTSEnumerateSessionsW                                               |
| >                                                                     |
| > DWORD CountSessionStructs; //Extracted used inside                  |
| > WTSEnumerateSessionsW                                               |
|                                                                       |
| DWORD dwCountGenerationAndStorage; //Increased per new execution for  |
| new user };                                                           |
+-----------------------------------------------------------------------+
```

At this point of execution, if some checks in terms of OS version and
current SID are met successfully, Qakbot will proceed to call
WTSEnumerateSessions() to enumerate Sessions on an RDP server.

In this sense, NetUserEnum first uses the server name parameter NULL, so
it spreads to the current computer accounts only. After this process has
finished, it will proceed to call NetGetDCName to get the primary domain
controller's name, and from this point, do some lateral movement in the
current network.

Let\'s describe how this is done:

1.-It will look up the account name and SID for one specific account
with LookupAccountNameW for the local computer/network.

2.-Will generate a random dll with the method and operations***.***

(***Storage::GenerateNewDllandRegistryPaths)***

In this sense, additionally to the expected behavior of the funcion
already described in terms of configuration storage, it will proceed to
build a proper regsvr32 path for execution.

> ![](.//media/image26.png)
>
> ![](.//media/image27.png)

The full command line to work with regsvr32 with the randomly generated
dll name is:

***regsvr32.exe -s \\ \<full dll path to random dll\> \\***

Now, for both remote sessions and the other accounts in the current
computer, once that regsvr32 command line has been generated, it will be
used to create a process with

CreateProcessAsUserW, where WTSQueryToken and EqualSID are used to check
for the SID.

> ![](.//media/image28.jpg)
>
> ![](.//media/image29.jpg)
After this process has finished, it writes the regsvr32 in:

\<CurrentSidString\>\\\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run

> ![](.//media/image30.jpg)

The main purpose of this is to achieve persistence after reboot, for
each account where this is set.

After the execution of the spreader component has finished, Qakbot will
generate a buffer long enough to store the current command line string
format:

***\"%s\\system32\\schtasks.exe\" /Create /RU \"NT AUTHORITY\\SYSTEM\"
/tn %s /tr \"%s\" /SC ONCE***

***/Z /ST %02u:%02u /ET %02u:%02u***

Some of the most relevant values in the sense of this string format
include for example the time, which is calculated for each case with
GetLocalTime, the Windows directory path and another regsvr32 command
line which passes the container path. ***(regsvr32 -s \\\\\\\<path
executable\>\\\\\\)***

Do you remember the ***indexCryptoStruct*** that I mentioned in the
previous section?.

This is checked here before the generation of the string format.
Additionally, using the container executable path, the in-memory
container buffer is written to it, if it exists.

***(Execution::GeneratePayloadsAndRegsvrInstanceDependingCheck)***

> ![](.//media/image31.jpg)

*(Important: The screenshot above is from a labeled sample from March,
nowadays the argument passed to the labeled function
String::WcharStringDecryptWrapper1 for decrypting the command line
format is 0x338 instead of 0x1f8. The calculations are still the same as
of June.)*

Another possibility based on the m\_checks\_sid\_option is that Qakbot
will proceed to try to build some command line to spawn a certain
process, but, as of today (6/16/2022), verified both statically and
dynamically, the string decryption for 0x3c0 for wchar\_t is instead
generating some broken exclusions path.

> ![](.//media/image32.jpg)
(Same as above, the core functionality has not changed, the main
difference is the string decryption wrapper, which in this case is 0x164
for the argument, but as of June, it is 0x3c0)

Generally, after I have described these two options of execution, the
last details to take in mind are these:

*- Detecting for attributes of the cfg file for the current executable
that has loaded the dll stager. This way it knows it exists, it proceeds
to delete it.*

*- Update the members depending on the values initialized inside the
execution of functions described in pervious chapter*

Once all these functions, there is no remarkable functionality to be
highlighted, therefore let\'s move on to the ***Execution::ThirdStage***
function, to describe the most important functionality for those that
want to create C2 emulators.

### Third stage function: Regsvr32 instances, more detections and function entries.

Let\'s describe the most important features that this "third stage" of
the stager has to offer, below you can see roughly how it looks after a
lot of annotation.

> ![](.//media/image36.png)

### Different methods to trigger regsvr32 execution, and injector based on extracted data:

On the other hand, I think it\'s important to describe some of the
usages that Qakbot has for spawning regsvr32 with payload dlls.

***Execution::GeneratePayloadsAndRegsvrInstanceDependingCheck?***

This function is core to this purpose, because all the methods mentioned
next will use it one way or another. There are two versions of the usage
of the function we need to be aware of.

The first case just creates a regsvr32 instance with some random dll
name which is based on tick count, as you can see below
(mersenne\_tick\_count\_info). *This can be achieved using either the
run persistence key or scheduled tasks persistence, depending on the
current privileges.*

***We will see the second usage when we look at the vectored exception
handling case.***

> ![](.//media/image37.png)

The main command line string format for scheduled tasks is:
***schtasks.exe /Create /RU \"NT AUTHORITY\\SYSTEM\" /SC ONSTART /TN %u
/TR \"%s\" /NP /F,*** where the first value in the string format is a
random number and the second one is the regsvr32 command line generated.

Window Classes are also used for triggering the regsvr32 execution. In
this sense, a Window is created with CreateWindowExA. Then once the
Window has been created it is hidden through SW\_HIDE using ShowWindow.
GetMessageA is used to retrieve for dispatching importante messages,
where TranslateMessage and DispatchMessageA help in this purpose.

> ![](.//media/image38.png)

The most important call in this process is RegisterClassA, since it
describes the WNDCLASSA that it is used for dispatching the messages. In
this sense, the windows dispatcher function will proceed to execute the
regsvr32 instance with methods already mentioned if the first additional
parameter (*wParam*) is the same as 4 or the MessageWindow (uMsg) value
is equal to 0x11.

> ![](.//media/image39.jpg)

**AllocConsole()** and ***SetConsoleCtrlHandler*** are also misused for
this purpose.

In this sense, the Handler function checks if **CTRL\_SHUTDOWN\_EVENT**
is set, if this is the case, it will proceed with the same function
described in the other method, spawning a regsvr32 instance.

As you can see, there is an additional check for WinSta0, using
GetProcessWindowStation and ***GetUserObjectInformationW***, this is
because processes started by a logged-on user are associated with the
Winsta0 desktop.

> ![](.//media/image40.png)
>
> ![](.//media/image41.jpg)
>
> ![](.//media/image42.png)

Vectored Exception Handling is also used for this purpose. In this
sense, if the exception matches one of the certain specific exception
codes, then the creation of a new regsvr32 instance is accomplished as
well.

For this case, if the ***HookEntrypoint*** method has been used, it will
be generated either a dll or exe path in the same folder as the
container executable, writing the file from a different file buffer
generated one of the ***function\_entry*** cases, setting the
appropriate persistence method, and generating a proper regsvr32 command
line, finally being spawned with CreateProcessW.

If a different method is used, then it will just generate a regsvr32
command line and spawn it.

After this has been executed, the event checked in the second stage is
sent and additionally, it will set the flag that ends the execution of
the ***third stage timed execution***, which we will see in the ***next
section.***

> ![](.//media/image43.png)
>
> Method for ***entrypoint hooking***
>
> ![](.//media/image49.jpg)
>
> Method for all other cases ***(Service and direct call)***

I want to additionally mention that Qakbot also implements an injector,
using a similar method of entrypoint hooking like the one described
before.

The main difference in this case, it\'s that the HookFunction goes
further than just fixing the IAT and API hashing for resolving the
functions to use: It\'s in charge of aspects such as properly mapping
the sections and executing the entrypoint of the payload directly,
resembling more to a proper manual mapping of the dll.

(The function is labeled as ***Injection::InjectorFromDataInresources***
inside the

***Execution::ThirdStage*** image at the start of this section)

In this specific case, the main buffers are mainly extracted from the
resources of a DLL that uses the computer info hash for building the
path.

(The core data of this DLL seems to be stored through the execution of
methods described in section

1.6, so how this is done is beyond the scope of this paper.)

Here are some images that describe some of the capabilities of the
injector:

> ![](.//media/image50.jpg)

### Named pipes communications, thread management, and timed execution based on configuration, with some more process detection.

First of all, it is important that we start describing how Qakbot
achieves the execution of multiple threads and the methods that involve
c2 functions.

The core structure for this purpose is:
```
+-----------------------------------------------------------------------+
| struct ThreadHandler                                                  |
|                                                                       |
| {                                                                     |
|                                                                       |
| > HANDLE hThread; DWORD ThreadID;                                     |
| >                                                                     |
| > void\* pFunctionToExecute; void\* pArgumentsBuffer;                 |
| >                                                                     |
| > DWORD dwBytesArguments;                                             |
| >                                                                     |
| > DWORD dwEndThreadFlag; //Checked before calling destructor of       |
| > Thread Handler instance. DWORD dwResultFunction; //return value     |
| > from function. HANDLE hMutexThread;                                 |
|                                                                       |
| };                                                                    |
+-----------------------------------------------------------------------+
```

Qakbot supports a maximum of 128 of these ThreadHandler structs, which
are used for additional functionality like anti-analysis mechanisms or
named pipe communications.

Inside the named pipes communications routine, we will find
functionality that leads to ***the execution of methods (function
entries) where the C2 communications functions are also in.***

> ![](.//media/image54.jpg)

To get further into this topic, it is necessary that we touch named
pipes communications. The main method that this is done is through
ConnectNamedPipe and ReadFile, where the buffer read is 0x80000,
therefore the most appropriate way to think in terms of this buffer is
that it is "serialized" for read/write.

> ![](.//media/image58.jpg)

Qakbot has many options depending on the wOption inside the IPCPacket,
but we are only interested when wOption is 7. In this specific case, a
buffer for pointers are specifically allocated and then passed as
arguments to a function that executes methods based on IDs.

```
+-----------------------------------------------------------------------+
| \#define MAX\_SIZE\_BUFFER 524280 struct IPCPacket                    |
|                                                                       |
| {                                                                     |
|                                                                       |
| > WORD wOption; //Option to be executed in handler.                   |
| >                                                                     |
| > WORD wReserved; //Initialized as 3, maybe number of members to      |
| > Initialize, not used in                                             |
|                                                                       |
| handler.                                                              |
|                                                                       |
| > DWORD dwSizeToSend; //Size of the buffer to send.                   |
| >                                                                     |
| > BYTE Buffer\[MAX\_SIZE\_BUFFER\]; //Buffer used for read/write in   |
| > communications.                                                     |
|                                                                       |
| };                                                                    |
+-----------------------------------------------------------------------+
```

> ![](.//media/image59.jpg)

Inside the function **Execution::ExecuteFunctionEntry**, different
functions can be executed depending on ID, having in mind certain
conditions.

The main structure for each function entry which is used in this case
is:

> struct function\_entry
>
> {
>
> WORD FunctionID; WORD checkNoThread; void \*functionPointer;
>
> };
>
> ![](.//media/image60.jpg)

This **Execution::ExecuteFunctionEntry** function can be both used in
the IPC handler described or in another function that will execute
functions based on time. *(There are other functions that can call it,
but they are out of scope for this section, e.g, called inside one
function entry itself)*

The structures for the timed execution are these and the function is
labeled as

```
+-----------------------------------------------------------------------+
| struct ExecuteTimeStruct                                              |
|                                                                       |
| {                                                                     |
|                                                                       |
| > TimeExecution \*timeContext;                                        |
|                                                                       |
| };                                                                    |
|                                                                       |
| struct TimeExecution {                                                |
|                                                                       |
| > DWORD dwTimingCount; //Check for current time, compare it if it\'s  |
| > more so it fails.                                                   |
| >                                                                     |
| > DWORD dwUnknown; //Set as 0 in the constructor, use is not clear    |
| > using xrefs.                                                        |
| >                                                                     |
| > DWORD FunctionID; //Function ID to execute, if pointer does not     |
| > exist. void\* functionPointer; //Function to execute. timeval       |
| > dwTimeSetExec; //Set tv\_sec and tv\_usec for currTime timeval      |
| > struct.                                                             |
|                                                                       |
| DWORD dwSecondsCheckTiming; //Used as check inside one function, most |
| likely seconds. DWORD bExecutedId; //Set as one after executing the   |
| function. };                                                          |
+-----------------------------------------------------------------------+
```

> ![](.//media/image64.jpg)

***ExecuteTimeStruct*** is the array of structures used for executing
functions based on time configuration. The format for the timed
configuration storage looks like this:

***\"%u;%u;%u\",
ExecutionFunctionTable.timeContext\[k\].dwTimingCount,***

> ***ExecutionFunctionTable.timeContext\[k\].FunctionID,***
>
> ***ExecutionFunctionTable.timeContext\[k\].dwTimeSetExec.tv\_sec);***

And the separation for each entry is the ***"\|"*** character, so it
roughly looks like this: ***%u;%u;%u \| %u;%u;%u \| %u;%u;%u \| .....***

It is important to mention that for this specific case, the functionID
used requires that the methods called do not pass arguments, this is
different for the IPC case, where this is possible.

The while loop of timed execution ends when one flag is set in mainly
through one regsvr32 execution method or in one IPC packet option. The
flag can be seen in the paper as ***dwSpawnedRegsvrEndStager***

This check is presented at the image in the start of the section as
**Execution::CheckEventEndStagerFunction()**

On the other hand, I consider important to describe the detection of
additional tooling at this point, which are:
```

***frida-winjector-helper-32.exe;frida-winjector-helper-64.exe;tcpdump.exe;windump.exe;ethereal
.exe;wireshark.exe;ettercap.exe;rtsniff.exe;packetcapture.exe;capturenet.exe;qak\_proxy;dump
cap.exe;CFFExplorer.exe;not\_rundll32.exe;ProcessHacker.exe;tcpview.exe;filemon.exe;procm
on.exe;idaq64.exe;PETools.exe;ImportREC.exe;LordPE.exe;SysInspector.exe;proc\_analyzer.ex
e;sysAnalyzer.exe;sniff\_hit.exe;joeboxcontrol.exe;joeboxserver.exe;ResourceHacker.exe;x64d
bg.exe;Fiddler.exe;sniff\_hit.exe;sysAnalyzer.exe***
```

```
+-----------------------------------------------------------------------+
| struct ToolingDetection //Struct made for detecting additional        |
| tooling in third stage                                                |
|                                                                       |
| { char \*\*ptrProcessesToDetect; //ptrs to strings of process names   |
| to detect. int dwNumberOfProcessesToDetect; //Number processes to     |
| detect.                                                               |
|                                                                       |
| char \*pPcapModuleString; //wpcap.dll decrypted str ptr stored here.  |
| };                                                                    |
+-----------------------------------------------------------------------+
```

> ![](.//media/image65.png)


If any of the processes listed above is found, the main check that is
done using

***CreateToolhelp32Snapshot, Module32First and Module32Next***, is in
terms of whether the module ***wpcap.dll*** is loaded inside of it. This
entire function is executed in a new thread.

If this is found, it will sleep for 1000 milliseconds each time it finds
it, using SleepEx, where the bAlertableState is used, so it can end
through time out or when I/O completion callback function occurs.

All the descriptions above are most of the core functionality that
people should be aware of, but there are some more details that
compliment some of the information missing during this paper, so let\'s
describe them as well.

Additional structures and details for reverse engineering Qakbot:
-----------------------------------------------------------------

These are some more important details that can be helpful for any
reverse engineer that wants to mess around more around the main Qakbot
dll stager.

My general advice is to look at the functions entries described before,
some of these functions contain the methods we need to be aware of for
doing proper C2 emulation, including the infamous sysinfo struct that is
constantly changing.

The best way to deal with the network communications could be doing some
binary rewriting

(specially to spot which parts of the functions are creating the JSON
and encrypting it)

**(I recommend looking at the Kaspersky report of this malware [[(link
here)](https://securelist.com/qakbot-technical-analysis/103931/)]{.underline}
for a general description on how this is done.)**

It describes the communication very well, and there are even more
sources, so just look them up and reverse engineer the proper methods as
well if you are interested in creating your own emulator)

C2 IPs used for proper communications inside the resources:
===========================================================

Inside one of the c2 handlers of the stager, we can describe the
structures that contain the IPs.

The main IP config structure looks like, which includes both registry
and resources retrieved C2 servers.
```
+-----------------------------------------------------------------------+
| struct custom\_ip                                                     |
|                                                                       |
| { in\_addr IP\_c2;//Proper C2 ip to be used.                          |
|                                                                       |
| > u\_short sin\_portC2; //Port which is manipulated in the            |
| > constructor.                                                        |
|                                                                       |
| };                                                                    |
|                                                                       |
| struct IpConfig //IP config used for proper communications.           |
|                                                                       |
| {                                                                     |
|                                                                       |
| > BYTE bCheckValidConfig; custom\_ip IPStruct;                        |
| >                                                                     |
| > BYTE padding\[20\]; //NULL bytes in each entry.                     |
|                                                                       |
| };                                                                    |
+=======================================================================+
| struct \_\_unaligned IpConfigBuffer //Buffer stored in registry and   |
| resources.                                                            |
|                                                                       |
| {                                                                     |
|                                                                       |
| > BYTE bValidStructIp; //Checked before decoding the values.          |
| >                                                                     |
| > DWORD ipEncoded; //IP encoded.                                      |
| >                                                                     |
| > WORD port; //Port encoded.                                          |
|                                                                       |
| };                                                                    |
+-----------------------------------------------------------------------+
```
This custom\_ip struct is used in one function with inet\_ntoa ***(which
was in the IAT of the binary, at least until March 2022).***

Of course, since there are multiple IPs stored for comms, an array of
structs of type ***IpConfig*** is used.

> ![](.//media/image66.png)

For successful extraction, you need to rewrite the operation related to
the ports in a Python script, mainly.

This structure is used in one method inside one function entry of the
methods already described above, so it is important that from there you
reverse engineer the rest.

Additionally, the communications are done through JSON so if you want an
additional challenge you can reverse the different structures/objects
employed for building it.

Corrupting container executable in disk and generating file buffers.
--------------------------------------------------------------------

We saw earlier in the second stage and third stage function that Qakbot
writes files to disk from a buffer of memory, but from where and how is
it done?

There are two main buffers used but both of them use the following
structure.
```
+----------------------------+
| struct fileBuffer          |
|                            |
| {                          |
|                            |
| > BYTE \*pFileBuffer;      |
|                            |
| DWORD dwFileBufferSize; }; |
+----------------------------+
```
The first buffer which is commonly used for all the functions
previously, such as

***Execution::GeneratePayloadsAndRegsvrInstanceDependingCheck,*** is
copied from the container executable, while the second case will use the
buffer obtained through one of the **function\_entry** methods already
described. ***(You will notice both structures when you xref it).***

It is important to mention how this ***fileStruct*** structure is used
for the ***container executable buffer,*** in the ***HookEntrypoint***
case before the proper execution state is selected in the second stage
function.

First of all, the first 1024 bytes will be read from the current
executable container, and then it will proceed to read armstream.dll
from the ***system32*** directory.

Once this has finished, if the size is more than 1024, it will proceed
to copy from the next 1024 bytes from the start for armstream.dll, to
this new payload buffer.

Next, it will proceed to write the container executable in disk until it
reaches the maximum size of 4096

> ![](.//media/image70.png)

Because it is written on disk, it makes sense the container executable
is corrupted and needs to be rewritten in other instances.

Prevention of infection in CIS countries:
-----------------------------------------

This function will be used both in the **SecondStage** and
***ThirdStage*** function, you will notice it 2 times when reversing the
stager.

The main operation is set through GetKeyboardLayoutList(), and will
verify for specific IDs.

> ![](.//media/image71.jpg)

The main keyboard layout IDs detected this way are found in CIS
countries, and these are:
```
00000419,0000041a,00000422,00000423,00000428,0000042b,0000042c,00000437,0000043f,00000
440,00000442,0000081a,0000082c,00000843,00000c1a,0000201a,00010419,0001042b,0001042c,0
0010437,00020419,00020422,0002042b,00020437,0003042b,00030437,00040437.

In terms of keyboards detected, we can find: Russian, Croatian,
Ukrainian, Belarusian, Tajik,

Armenian Eastern, Georgian, Kazakh, Kyrgyz Cyrillic, Turkmen, Serbian
(Latin), Azerbaijani Cyrillic,

Uzbek Cyrillic, Serbian (Cyrillic), Bosnian (Cyrillic), Russian
(Typewriter), Armenian Western,

Azerbaijani (Standard), Georgian (QWERTY), Russian - Mnemonic, Ukrainian
(Enhanced), Armenian Phonetic, Georgian (Ergonomic), Armenian
Typewriter, Georgian Ministry of Education and Science Schools, Georgian
(Old Alphabets).
```