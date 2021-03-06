![alt text](media/powergrr.png "PowerGRR Logo")

# PowerGRR - PowerShell Module for GRR API

PowerGRR is a PowerShell module for working with the GRR API working on Windows, macOS and Linux. 

Please see [Command Documentation](docs/PowerGRR.md), [Wiki](https://github.com/swisscom/PowerGRR/wiki) and
[CHANGELOG](CHANGELOG.md).

***
<!-- vim-markdown-toc GFM -->

* [What is PowerGRR?](#what-is-powergrr)
* [Requirements](#requirements)
    * [GRR server](#grr-server)
    * [PowerShell](#powershell)
    * [Certificate authentication](#certificate-authentication)
    * [Known issues on non-Windows platforms](#known-issues-on-non-windows-platforms)
* [Installation](#installation)
* [Configuration](#configuration)
* [Usage](#usage)
    * [Import](#import)
    * [Authentication](#authentication)
    * [Cmdlets](#cmdlets)
    * [Help](#help)
    * [Example](#example)
* [Contributing](#contributing)

<!-- vim-markdown-toc -->
***

## What is PowerGRR? 

PowerGRR is a **PowerShell module for working with the
[GRR](https://github.com/google/grr) API working on Windows, macOS and
Linux**. GRR Rapid Response is an incident response framework focused on
remote live forensics. The module allows working with flows, hunts, labels,
artifacts, approvals and the search feature. Furthermore, it allows **working
with the computer names instead of the GRR internal client id**. This makes
handling and working with other tools more easy because often you just have
the computer names. PowerGRR also enables you to easily document your work in
text form which is then directly reusable by others.

PowerGRR creates a comfortable, cli-based workflow for incident response.
PowerShell is installed on every Windows workstation and working directly with
PowerShell objects enables you to sift quickly through flow and hunt data.
This object-oriented approach gives you a fast way to analyze output
within PowerShell, e.g. get all unique registry paths from a hunt or show a
list of unique clients where a file was found.

Some of the use cases where PowerGRR could speed up the work:
* Start a flow on one or multiple clients and get flow results as PowerShell
    object for easier filtering.
* Create and start a new hunt and get the hunt info or results as PowerShell
    objects.
* [Create a hunt or a client approval request and wait until they get valid](https://github.com/swisscom/powergrr/wiki#commands-for-using-the-grr-approval-system).
* Add or remove a label on one or multiple clients based on a list of computer
    names.
* Add artifacts to or remove artifacts from the GRR artifact repository.
* List hunts, artifacts, client or hunt approvals, labels and clients and
  filter them in different ways.
* Build [IR scripts for common forensic workflows and start multiple hunts or
    flows in one shot using multiple cmdlets inside a PowerShell script](https://github.com/swisscom/PowerGRR/wiki/Live-response-collection-script).

The following flow types are available for hunts and flows and the target group
is chosen based on labels or the OS. See also [command
help](https://github.com/swisscom/PowerGRR/blob/master/docs/Invoke-GRRFlow.md#-flow)
for the available flow types.
* Netstat, ListProcesses, FileFinder, RegistryFinder, ExecutePythonHack, ArtifactCollectorFlow, YaraProcessScan

**Repo Structure**

| Name              | Description                                                        |
| ----------------  | ------------------------------------------------------------------ |
| docs\             | Markdown documentation                                             |
| en-us\            | With playPS created PowerShell helpfile (use `help <command>`)     |
| test\             | Pester tests (for using with Invoke-Pester)                        |
| BUILD.md          | Build instructions for ctags, playPS and Pester                    |
| CHANGELOG.md      | Changelog of the project                                           |
| PowerGRR.psd1     | PowerGRR module description                                        |
| PowerGRR.psm1     | PowerGRR module file                                               |
| tags              | ctags file for PowerGRR                                            |

## Requirements

### GRR server

To be able to use all PowerGRR commands, one must use the current version of GRR.
Some API calls, like starting a hunt or uploading or removing an artifact, are only
working with current versions of GRR and not with the previous versions.

### PowerShell

Windows PowerShell or [PowerShell
Core](https://github.com/PowerShell/powershell) is required in order to run
PowerGRR. See [PowerShell](https://msdn.microsoft.com/en-us/powershell) for
more information about PowerShell itself.

Windows has Windows PowerShell installed by default - so nothing to do here. 

For **macOS and Linux see [PowerShell Package installation
instructions](https://github.com/PowerShell/PowerShell/blob/master/docs/installation/linux.md)**
for information about how to get PowerShell Core installed. The installation is
fast and for most OSes there are ready to use packages and available
installation commands.

### Certificate authentication

If you are using certificate authentication, a PKCS12 certificate is required. You can
convert your existing certificate with the following openssl command:

```
openssl pkcs12 -export -in example.crt -inkey example.key -out certificate.p12
```

* Use the Windows certificate store and add a config option for the certificate 
issuer which certificate should be used
* Use client certificate files with the corresponding config option. Client certificate 
authentication with files works with _Windows PowerShell_ and since _v6.0.0-beta.6 
with PowerShell Core_ (@markekraus made the needed pull request - [#4546](https://github.com/PowerShell/PowerShell/pull/4546)).

If you are using certificate authentication with PowerShell Core on macOS, 
please see paragraph **Client certificate authentication using
PowerShell Core** _in section [Known issues on non-Windows
platforms](#known-issues-on-non-windows-platforms)_ below for further
information.

### Known issues on non-Windows platforms

Despite GRR related commands were tested on PowerShell Core on different OSes
(Windows, Linux, macOS), some issues exist on non-Windows platforms. _If you
run into other troubles with PowerShell Core or with non-Windows platforms,
please file an issue._

**PowerShell help**

The PowerShell help examples (`help <command> -Examples`) are not shown in
Ubuntu 16.04. See issue [#9](https://github.com/swisscom/PowerGRR/issues/9).

**Client certificate authentication using PowerShell Core**

The client certificate authentication on macOS is currently not supported 
in PowerShell Core release packages on Github (see PowerShell Core issue [#4650](https://github.com/PowerShell/PowerShell/issues/4650)).

## Installation

* Install PowerGRR from [PowerShell Gallery](https://www.powershellgallery.com/packages/PowerGRR/). [PowerShellGet](https://github.com/powerShell/powershellget) is required which is installed in PowerShell Core and since Windows PowerShell v5 by default. Only released versions are available there, see [CHANGELOG](CHANGELOG.md).

    ``` powershell
    # Inpsect
    Save-Module -Name PowerGRR -Path <path> 

    # Install
    Install-Module -Name PowerGRR -Scope CurrentUser

    # Update
    Update-Module -Name PowerGRR
    ```
   
   If you get an error, try using the following command to set PowerShell Gallery as one of your repositories:
   
   ``` powershell
   Register-PSRepository -Default
   ```
   
* Install PowerGRR from Github:

    * Clone or download the repo into your module path folder, usually
      _~\Documents\WindowsPowerShell\modules_ on Windows or
      _~/.local/share/powershell/Modules/_ on macOS (see _$env:PSModulePath_).
    * Clone or download the files to any other folder (could also be a share).
    * **Windows** Make sure to unblock the files when downloaded from the
        Internet by opening the properties page of the .psd1 and .psm1 files and
        checking "Unblock" at the bottom.

    The location changes how the module is imported.

## Configuration

1. Create a 'powergrr-config.ps1' in the profile folder (`$env:USERPROFILE` or
`$env:HOME`) or in the root folder of the module.
1. Set the config variables as needed. 
   * **[MUST]** _$GRRUrl_: GRR server's URL.
   * **[OPTIONAL]** _$GRRIgnoreCertificateErrors_: If set to $true certificate errors are ignored.
   * **[OPTIONAL]** _$GRRClientCertIssuer_:  If set, the client certificate
                    from the Windows cert store signed by the given issuer is used.
   * **[OPTIONAL]** _$GRRClientCertFilePath_: If set, the client certificate
                   file is used for the authentication. If you are using
                   PowerShell Core see section [certificate
                   authentication](#certificate-authentication) in
                   requirements above.

It's also possible to set these variables in the console.

**Example Config**

``` PowerShell
$GRRUrl = "https://grrserver.tld"
$GRRClientCertIssuer = "issuer of the certificate"
```

If you want to get crazy you could even use a config file file looking
like this if you need to constantly change the GRR config otherwise. You only
need to change the comment for the GRRUrl.

``` powershell
#$GRRUrl = "https://main-grrserver.tld"
$GRRUrl = "https://test-grrserver.tld"
$GRRIgnoreCertificateErrors = $( if ($GRRUrl -match "test") { $true } )
$GRRClientCertIssuer = $( if ($GRRUrl -match "main") { "certificate issuer" } )
```

## Usage

### Import

If PowerGRR was saved inside the module path run the following command:
```
Import-Module PowerGRR -force
```

If PowerGRR was saved outside the module path run the command:

```
Import-Module <path to module>\PowerGRR.psd1 -force
```

### Authentication

1. Store your GRR credentials for any subsequent PowerGRR command or otherwise
you will be prompted when running the commands. Either provide the credentials
with `-Credential` in each command or use the variable `$GRRCredential` to set
the credentials which then will be used without the need for supplying
`-Credential`.

```powershell
$GRRCredential = Microsoft.PowerShell.Security\get-credential
```

2. If you use client certificate authentication set the corresponding config
variable as described in [Configuration](#configuration) above. If you are
using PowerShell Core see section [certificate
authentication](#certificate-authentication) in requirements above.

### Cmdlets

Please see [docs](docs/PowerGRR.md) for the list of all available commands and the
[wiki](https://github.com/swisscom/PowerGRR/wiki) for further information how
you could use and combine the different PowerGRR commands.

Use the common parameters like _-WhatIf_ or _-Verbose_ for troubleshooting and to
see what the commands would do. _WhatIf_ is implemented for every function which
make any permanent change (e.g. start a flow, set a label, ...).

List available PowerGRR commands.

```powershell
get-command -Module PowerGRR
```

List all PowerGRR commands for flows.

```powershell
get-command -Module PowerGRR | sls flow
```

### Help

Use `help <command>` to get the help for a command.

```powershell
PS> help Get-GRRHuntInfo

NAME
    Get-GRRHuntInfo

OVERVIEW
    Get hunt info for a specific hunt.

SYNTAX
    Get-GRRHuntInfo [[-HuntId] <String>] [-Credential] <PSCredential> [-ShowJSON] [<CommonParameters>]
...
```

Use `help <command> -Examples` to get examples for a command.

```powershell
PS> help Get-GRRHuntInfo -Examples

NAME
    Get-GRRHuntInfo

OVERVIEW
    Get hunt info for a specific hunt.

    Example 1

    PS C:\> Get-GRRHuntInfo "H:AAAAAAAA" -Credential $cred
...
```

### Example

The following examples shows how you could combine the different PowerGRR functions
to quickly label some clients, start a flow against them or a hunt based on a label
and read the results. You can find more code snippets and ideas in the
[wiki](https://github.com/swisscom/PowerGRR/wiki) and see section
[help](#help) above how to use the help system in PowerShell.

Use `$GRRCredential` for setting the credentials before running the commands
and the parameter `-Credential` is not needed anymore for each command.

```powershell
# Read the client information to check LastSeenAt and the OSVersion
Get-GRRClientIdFromComputerName -ComputerName WIN-DESKTOP01,MBP-LAPTOP02,WIN-DESKTOP03,WIN-DESKTOP04 `
                                -Credential $creds
 
ComputerName    ClientId           LastSeenAt          OSVersion
------------    --------           ----------          ---------
WIN-DESKTOP01   C.aaaaaaaaaaaaaaaa 18.05.2017 15:48:17 10.0.10586
WIN-DESKTOP01   C.xxxxxxxxxxxxxxxx 03.04.2017 14:55:37 6.1.7601
MBP-LAPTOP02    C.bbbbbbbbbbbbbbbb 18.05.2017 15:49:12 16.6.0
WIN-DESKTOP03   C.dddddddddddddddd 11.03.2017 10:23:51 10.0.10586
WIN-DESKTOP04   C.eeeeeeeeeeeeeeee 11.03.2017 10:23:51 10.0.10586

# Set a label for multiple hosts during incident response with the parameter
# __ComputerName__
Set-GRRLabel -ComputerName WIN-DESKTOP01, WIN-DESKTOP03, WIN-DESKTOP04 -Label INC02_Windows `
             -Credential $creds

# or through the pipeline
"MBP-LAPTOP02" | Set-GRRLabel -Label INC02_macOS -Credential $creds

# Now you can work with that label within GRR UI or in the shell. Use
# -OnlyComputerName to only display the hostname instead of the full GRR client
# object
$clients = Find-GRRClientByLabel -SearchString INC01 -Credential $creds -OnlyComputerName

# Start a flow on the affected clients
$clients | Invoke-GRRFlow -flow RegistryFinder `
                          -key "HKEY_USERS/%%users.sid%%/Software/Microsoft/Windows/CurrentVersion/Run/*" `
                          -Credential $cred

# Get flow results - see output of specific flow ids. Using
# -OnlyPayload navigates directly to the payload section of the results
# within the returned GRR object
$ret = Get-GRRFlowResult -Credential $cred -ComputerName WIN-DESKTOP01 -FlowId "F:11111111" -OnlyPayload

# Show only the registry paths from the returned GRR object. Sometimes the
# output is base64 encoded. Get-GRRFlowResult decodes the string if
# possible. 
$ret.stat_entry.registry_data

# Alternative you can start a hunt against that label. The EmailAddress
# parameter is optional and notifies you about the first hit. The OnlyUrl
# parameter shows only the URL to the hunt.
$HuntId = New-GRRHunt -HuntDescription "Search for notepad.exe" `
            -Flow FileFinder `
            -path "c:\notepad.exe" `
            -MatchMode MATCH_ALL `
            -actiontype hash `
            -RuleType label `
            -Label INC01 `
            -EmailAddress your@email.tld `
            -Credential $creds `
            -OnlyUrl `
            -Verbose

# If needed request an approval
$ApprovalId = New-GRRHuntApproval -Credential $cred -HuntId H:AAAAAAAA -NotifiedUsers user1 `
                    -Reason "Hunting for notepad.exe - INC01" -OnlyId

# Start the hunt
Start-GRRHunt -Credential $creds -HuntId $HuntId

# Start the hunt after approval got within the given timeout
Start-GRRHunt -HuntId $HuntId -Credential $creds -Wait -ApprovalId $ApprovalId -TimeoutInMinutes 15

# Read hunt restuls
$ret = Get-GRRHuntResult -Credential $cred -HuntId $HuntId

# Inspect results
$ret.items

# Filter results as needed - e.g. see unique clients which were affected 
$ret.items.client_id | get-unique

# Get unique computer names based on the list of client ids
$ret.items.client_id | Get-GRRComputerNameFromClientId -Credential $cred | get-unique

# Get unique file paths from a file finder hunt
($ret.items.payload.stat_entry.aff4path).substring(31) | sort -u

# Remove the label if you don't use it anymore
$clients | Remove-GRRLabel -SearchString INC01 -$Credential $creds
```

## Contributing

See [CONTRIBUTING](CONTRIBUTING.md) for general guidelines and some inner
workings of PowerGRR.
