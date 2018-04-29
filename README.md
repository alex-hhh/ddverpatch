# About

Verpatch is a command line tool for adding and editing the version information
of Windows executable files (applications, DLLs, kernel drivers) without rebuilding the executable.

It can also add or replace Win32 (native) resources, and do some other modifications of executable files, such as stripping the debug information (pdb) path.

The project has been migrated to GitHub from the original [CodePlex project](https://ddverpatch.codeplex.com/), which in turn were a continuation of a [CodeProject article](https://www.codeproject.com/Articles/37133/Simple-Version-Resource-Tool-for-Windows) (with source) originally published in 2009.

## Note

These days Visual C++ has resource editor for free, so it is no really needed; but it might still come in handy, and the source code might be useful for reference.

## Thanks

Thanks to reviewers and contributors on CodeProject and [Codeplex](https://ddverpatch.codeplex.com/team/view).

# Usage

```
verpatch filename [version] [/options]
```

Where
 * filename : Any Windows PE file (exe, dll, sys, ocx...) that can have a version resource
 * version : One to four decimal numbers, separated by dots, ex.: 1.2.3.4<br>
   Additional text can follow the numbers; see examples below. Ex.: "1.2.3.4 extra text"

The command will set ERRORLEVEL 0 on success, otherwise errorlevel is non-zero.

**Note:** Verpatch modifies files in place, so please make copies of precious files.

## Main Options:

/va - creates a version resource. Use when the file has no version resource at all,
     or existing version resource should be replaced.
     If /va is not specified, verpatch will read and modify version resource from the file.

/s *name* "value" - add a version resource string attribute
     The name can be either a full attribute name or an alias; see below.
     See: *Known string keys in VS_VERSION_INFO resource* for list of known attribute **name**s.

/sc "comment" - add or replace Comments string (shortcut for /s Comments "comment")

/pv \<version\>   - specify the Product version
    where \<version\> arg has same form as the file version (1.2.3.4 or "1.2.3.4 text")

/high - when less than 4 version numbers, these are major numbers. By default, less than 4 numbers are minor numbers.


## Additional options

/fn - preserves Original filename, Internal name in the existing version resource of the file.

/langid \<number\> - language id for new version resource.<br>
     Use with /va. Default is Language Neutral.<br>
     \<number\> is combination of primary and sub-language IDs. ENU is 1033 or 0x409.

/vft2 num - specify driver subtype (VFT2_xxx value, see winver.h)
     The application type (VFT_xxx) is retained from the existing version resource of the file,
     or filled automatically, based on the filename extension (.exe->app, .sys->driver, anything else->dll)

/vo - outputs the version info in RC format to stdout.
     This can be used with /xi to dump a version resource without modification.
     Output of /vo can be pasted to a .rc file and compiled with rc.

/xi- test mode. does all operations in memory but does not modify the file

/xlb - test loopback mode. Re-parses the version resource after modification.

/rpdb - removes path to the .pdb file in debug information; leaves only the file name. This has no relation to the "original file name" attribute.

/rf #id file - add or replace a raw binary resource from file (see below)

/noed - do not check for extra data appended to the exe file (see below)


## Examples

```verpatch d:\foo.dll 1.2.33.44```

Sets the file version to 1.2.33.44.
The Original file name and Internal name strings are set to "foo.dll".
File foo.dll should already have a version resource (since /va not specified).

```verpatch d:\foo.dll 1.2.33 /high```

Sets three major 3 numbers of the file version. The 4th number not changed.
File foo.dll should already have a version resource.

```verpatch d:\foo.dll 33.44  /s comment "a comment"```

Replaces only two minor numbers of the file version and adds a comment.
File foo.dll should already have a version resource.

```verpatch d:\foo.dll "33.44 special release" /pv 1.2.3.4```

Same as previous, with additional text in the version argument.
Product version is also fully specified

```verpatch d:\foo.dll "1.2.33.44" /va /s description "foo.dll" /s company "My Company" /s copyright "(c) 2009"```

Creates or replaces version resource to foo.dll, with several string values

```verpatch d:\foo.dll /vo /xi```

Dumps the version resource in RC format, does not update the file.
    
## Details

In "patch" mode (no /va option), verpatch replaces the version number in existing file 
version info resource with the values given on the command line.
The version resource in the file  is parsed, then parameters specified on the command line are applied.

If the file has no version resource, or you want to discard the existing resource, use /va switch.

Quotes surrounding arguments are needed for the command shell (cmd.exe), for any argument that contains spaces.
Also, other special characters should be escaped (ex. &, |, and ^ for cmd.exe).
Null values can be specified as empty string ("").

The command line can become very long, so you may want to use a batch file or script.
See the example batch files.

The Version argument can be specified as 1 to 4 dot separated decimal numbers.
If the switch /high is not specified and less than 4 numbers are given,
they are considered as minor numbers.
The major version parts are retained from existing version resource.
For example, if the existing version info block has version number 1.2.3.4
and 55.66 specified on the command line, the result will be 1.2.55.66.

If the switch /high is specified and less than 4 numbers are given,
they are considered as higher numbers.
For example, if the existing version info has version number 1.2.3.4
and 55.66 /high specified on the command line, the result will be 55.66.3.4.

Additional suffix can follow the version numbers, separated by a dash (-) + or space.
If the separator is space, the whole version argument must be enclosed in quotes.

Verpatch ensures that the version numbers in the binary part
of the version structure and in the string part (as text) are same,
or the text string begins with the same numbers as in the binary part.
But the string representation of file or product version will not be displayed by Windows Explorer in Vista or newer. The file version displayed will always have 4 parts and no suffix.

The /high switch has been added to support the *"Semantic Versioning"* syntax
as described [here](http://semver.org).
The "Semantic versioning", however, specifies only 3 parts for the version number,
while Windows version numbers have 4 parts.
Switch /high allows 3-part version numbers with optional "tail" separated by '-' or '+'.

By default, Original File Name and Internal File Name are replaced by the actual filename.
Use /fn to preserve existing file names in the version resource.

String attribute names for option /s must be language-neutral, 
not translations (example: PrivateBuild, not "Private Build Description").
See [here|Known string keys in VS_VERSION_INFO resource] for the list of known attributes names and their aliases.
The examples above use the aliases.

String arguments for File version and Product version attributes are handled
 in a special way, the /s switch should not be used to set these:
 - The File version can be specified as the 2nd positional argument only
 - The Product version can be specified using /pv switch


## Extras

### Adding binary resource from file

The /rf switch adds a resource from a file, or replaces existing resource with same type and id.

The argument "#id" is a 32-bit hex number, prefixed with #.
Lowest 16 bits of this value are resource id; can not be 0.
Next 8 bits are the resource type: one of RT_xxx constants in winuser.h, or user defined.
If the type value is 0, RT_RCDATA (10) is assumed.
High 8 bits of the #id arg are reserved0.
The language code of resources added by this switch is 0 (Neutral).
Named resource types and ids are not implemented.
The file is added as opaque binary chunk; the resource size is rounded up to 4 bytes
and padded with zero bytes.

### Removing path to the debug information file (pdb)

This was added for these who consider the full path to the pdb file stored in the executable sensitive information (or use embarrassing names for their project directories).

Option /rpdb changes the path to the .pdb file to "x:\" followed by the original PDB file name. This will not break the association of the PDB file with the executable. It still can be used to debug, if the actual path is specified to the debugger.


### Preserving extra data appended to executable

Verpatch detects extra data appended to executable files, saves it and appends 
again after modifying resources. Command switch /noed disables checking for extra data.
The extra data is any data following the last section described in the PE header.

Such extra data is used by some installers, self-extracting archives and other applications.
However, the way we restore the data may be not compatible with these applications.
Please, verify that executable files that contain extra data work correctly after modification.
Make backup of valuable files before modification.

Note: digital certificates appended to the file cannot be reliably removed by /noed; this issue may be fixed in next releases. For now, use "delcert" utility (google) to remove certificates from a file before using verpatch on it.

## Version resource (VS_VERSION_INFO)

Below are listed well known string attributes of Version resource and shortcut names (aliases) that Verpatch understands.

The aliases in the right column can be used with /s switch,
in place of +language-neutral+ (LN) attribute names. 
Attribute names are not case sensitive.


| Lang.Neutral name | English translation        | Aliases                                 | Note |
| ----------------- | -------------------------- | --------------------------------------- | ---- |
| Comments          | Comments                   | comment                                 |      |
| CompanyName       | Company                    | company                                 |      |
| FileDescription   | Description                | description, desc                       | (E)  |
| FileVersion       | File Version               |                                         | (*1) |
| InternalName      | Internal Name              | title                                   |
|                   | Language                   |                                         | (*2) |
| LegalCopyright    | Copyright                  | copyright, (c)                          | (E)  |
| LegalTrademarks   | Legal Trademarks           | tm, (tm)                                | (E)  |
| OriginalFilename  | Original File Name         |                                         |      |
| ProductName       | Product Name               | product                                 |      |
| ProductVersion    | Product Version            | pv, productversion, productver, prodver | (*1) |
| PrivateBuild      | Private Build Description  | pb, private                             |      |
| SpecialBuild      | Special Build  Description | sb, build                               |      |
| OleSelfRegister   | -                          | -                                       | (A)  |
| AssemblyVersion   | -                          | -                                       | (N)  |

Notes

*1: FileVersion, ProductVersion should not be specified with /s switch.
See the command line parameters above.
The string values normally begin with same 1.2.3.4 version number as in the binary header,
but can be any text. Explorer of WinXP also displays File Version text in the strings box.
In Win7 or newer, Explorer displays the version numbers from the binary header only.

*2: The "Language" value is the name of the language code specified in the header of the
 string block of VS_VERSION_INFO resource (or taken from VarFileInfo block?)
It is displayed by Windows XP Explorer.

E: Displayed by Windows Explorer in Win7

A: Intended for some API (OleSelfRegister is used in COM object registration)

N: Added by some .NET compilers. This version number is not contained in the
   binary part of the version struct and can differ from the file version.
   To change it, use switch /s AssemblyVersion [value]. Note: this will not
   change the actual .NET assembly version.

## UAC (administrative privileges)

Verpatch does not require any administrator rights and may not work correctly if run elevated.

## Character encoding

Working with characters from extended character sets in Command Prompt and Batch files
might give you some challenges. For instance if you want to create a Batch script executing
Verpatch to enter the copyright symbol © into the copyright resource (`/s copyright "© 2017"`),
you might end up with the trademark symbol ® instead! The issue is that the command line
interpreter uses code page 850, the so-called OEM code page, which is different from the
Windows ANSI/Latin 1 character set you probably (in western Europe) would save your
Batch file in. You could of course just use the ASCII equivalent "(C)" instead,
but setting the code page 1252 (`chcp 1252 > nul`) in the beginning of your batch
script will make the copyright symbol show as intended. 

# Building

Just get the source and build in a compatible Visual Studio.

The source can be compiled with VC 2008 and newer, and projects for Visual C++ 2017, 2013 and 2008 are included:
* The VC2017 project is the files verpatch(vs2017).sln and verpatch(2017).vcxproj. The application are built with platform toolset v141, so it can run on Windows Vista (and newer).
* The VC2013 project is the files verpatch.sln and verpatch.vcxproj. The application will be built with platform toolset v120_xp, so it can run on Windows XP SP3 (and newer).
* The VC2008 project is the files verpatch(vs2008).sln and verpatch.vcproj. The built application can run on Windows 2000 (and newer).

We prefer to link statically with the runtime library rather than use runtime DLLs, to "streamline user experience", as MS people use to say. You can link with DLL runtime, if you wish.

The original source for [CodeProject article](http://www.codeproject.com/KB/install/VerPatch.aspx) demonstrates use of the UpdateResource API and imagehlp.dll.
It does not demonstrate a good use of C++, good coding style or anything else.

# Hacking

Issues and pull requests are welcome.

# Links/References

MSDN on VERSIONINFO resources

https://msdn.microsoft.com/en-us/library/windows/desktop/aa381058(v=vs.85).aspx

Semantic versioning
http://semver.org

Raymond Chen, 2006 / The evolution of version resources - 32-bit version resources
http://blogs.msdn.com/b/oldnewthing/archive/2006/12/21/1340571.aspx

Raymond Chen, 2006 / The evolution of version resources – corrupted 32-bit version resources (continuation of the previous)
https://blogs.msdn.microsoft.com/oldnewthing/20061222-00/?p=28623

Raymond Chen, 2006 / The first parameter to VerQueryValue really must be a buffer you obtained from GetFileVersionInfo
https://blogs.msdn.microsoft.com/oldnewthing/20061226-06/?p=28603

Raymond Chen, 2012 / Why doesn’t the Version tab show up for very large files?
https://blogs.msdn.microsoft.com/oldnewthing/20120418-00/?p=7833/

Raymond Chen, 2011 / PE resources must be 4-byte aligned, but that doesn’t stop people from trying other alignments
https://blogs.msdn.microsoft.com/oldnewthing/20110609-00/?p=10463

Raymond Chen, 2012 / Why does the VerQueryValue function give me the wrong file version number?
https://blogs.msdn.microsoft.com/oldnewthing/20120315-00/?p=8093 - about MUI satellite DLL  redirection

Search in the blog of Raymond Chen for everything related to version resources.

# Versions

## 1.0.15 (25-Oct-2016)

## 1.0.14 (04-Aug-2013)

Fixed issue with ANSI VerQueryValue truncating last character.

## 1.0.12 (21-Jun-2013)

## 1.0.10 (02-Sep-2012)

Included build configuration for VC2005, which did not know about the UAC manifest configuration (requestedExecutionLevel), so it had to be added via the *additional manifest file* configuration
to avoid Windows 7 (erroneously) deciding that Verpatch should run elevated. With later upgrade
to VC2008 this workaround could be removed, as there was a specific AsInvoker project setting.

# Known issues and TODO

 - Does not work on old PE files that have link version 5.x (before VC6?)
   No known workaround; this seems to be limitation of Windows UpdateResource API.
   Since the UpdateResource API is part of Windows, its behaviour may differ on
   different Windows releases. On Win7 SP1 you may get better results than on WinXP.
   
 - Import of version resource does not work if it is not encoded in UTF-16.

 - Does not work on files signed with digital certificates (TO DO: Warn and remove certificate)
   Until we do this, certificates can be removed with 3rd party delcert tool.

 - A second version resource may be added to a file that already has a version resource
   in other language. Switch /va won't help.
   TODO: Ensure that a file has only one version resource!
   
 - When verpatch is invoked from command prompt, or batch file, the string
   arguments can contain only ANSI characters, because cmd.exe batch files cannot be 
   in Unicode format. If you need to include characters not in current locale,
   use other shell languages that fully support Unicode (Powershell, vbs, js).
   
 - TODO: In RC source output (/vo), special characters in strings are not quoted;
   so /vo may produce invalid RC input.
   
 - The parser of binary version resources handles only the most common type of structure.
   If the parser breaks because of unhandled structure format, try /va switch to
   skip reading existing version resource and re-create it from scratch.
   Consider using WINE or other open source implementations?
   
 - option to add extra 0 after version strings : "string\0"
   (tentative, requested by a reader for some old VB code) 

 - For files with extra data appended, checksum is not re-calculated.
   Such files usually implement their own integrity check.

 - Switch /va does not prevent import of existing version resource. Revise.

 - When existing version string contains "tail" but the command line parameter does not,
   the tail is removed. In previous versions the tail was preserved.

 - Running Verpatch on certain executables (esp. built with GNU) produce corrupt file
   when run on WinXP SP3, but same binaries give good result when run on Win7 or 2008R2.
   (Improvement of UpdateResource API?)
