```sql
// Translated content (automatically translated on 16-04-2025 01:58:40):
event.type="Process Creation" and (endpoint.os="windows" and (((tgt.process.image.path contains "\wevtutil.exe" and (tgt.process.cmdline contains "clear-log " or tgt.process.cmdline contains " cl " or tgt.process.cmdline contains "set-log " or tgt.process.cmdline contains " sl " or tgt.process.cmdline contains "lfn:")) or ((tgt.process.image.path contains "\powershell.exe" or tgt.process.image.path contains "\pwsh.exe") and (tgt.process.cmdline contains "Clear-EventLog " or tgt.process.cmdline contains "Remove-EventLog " or tgt.process.cmdline contains "Limit-EventLog " or tgt.process.cmdline contains "Clear-WinEvent ")) or ((tgt.process.image.path contains "\powershell.exe" or tgt.process.image.path contains "\pwsh.exe" or tgt.process.image.path contains "\wmic.exe") and tgt.process.cmdline contains "ClearEventLog")) and (not ((src.process.image.path in ("C:\Windows\SysWOW64\msiexec.exe","C:\Windows\System32\msiexec.exe")) and tgt.process.cmdline contains " sl "))))
```


# Original Sigma Rule:
```yaml
title: Suspicious Eventlog Clearing or Configuration Change Activity
id: cc36992a-4671-4f21-a91d-6c2b72a2edf5
status: stable
description: |
    Detects the clearing or configuration tampering of EventLog using utilities such as "wevtutil", "powershell" and "wmic".
    This technique were seen used by threat actors and ransomware strains in order to evade defenses.
references:
    - https://github.com/redcanaryco/atomic-red-team/blob/f339e7da7d05f6057fdfcdd3742bfcf365fee2a9/atomics/T1070.001/T1070.001.md
    - https://eqllib.readthedocs.io/en/latest/analytics/5b223758-07d6-4100-9e11-238cfdd0fe97.html
    - https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wevtutil
    - https://gist.github.com/fovtran/ac0624983c7722e80a8f5a4babb170ee
    - https://jdhnet.wordpress.com/2017/12/19/changing-the-location-of-the-windows-event-logs/
author: Ecco, Daniil Yugoslavskiy, oscd.community, D3F7A5105
date: 2019-09-26
modified: 2023-07-13
tags:
    - attack.defense-evasion
    - attack.t1070.001
    - attack.t1562.002
    - car.2016-04-002
logsource:
    category: process_creation
    product: windows
detection:
    selection_wevtutil:
        Image|endswith: '\wevtutil.exe'
        CommandLine|contains:
            - 'clear-log '          # clears specified log
            - ' cl '                # short version of 'clear-log'
            - 'set-log '            # modifies config of specified log. could be uset to set it to a tiny size
            - ' sl '                # short version of 'set-log'
            - 'lfn:'                # change log file location and name
    selection_other_ps:
        Image|endswith:
            - '\powershell.exe'
            - '\pwsh.exe'
        CommandLine|contains:
            - 'Clear-EventLog '
            - 'Remove-EventLog '
            - 'Limit-EventLog '
            - 'Clear-WinEvent '
    selection_other_wmi:
        Image|endswith:
            - '\powershell.exe'
            - '\pwsh.exe'
            - '\wmic.exe'
        CommandLine|contains: 'ClearEventLog'
    filter_msiexec:
        # Example seen during office update/installation:
        #   ParentImage: C:\Windows\SysWOW64\msiexec.exe
        #   CommandLine: "C:\WINDOWS\system32\wevtutil.exe" sl Microsoft-RMS-MSIPC/Debug /q:true /e:true /l:4 /rt:false
        ParentImage:
            - 'C:\Windows\SysWOW64\msiexec.exe'
            - 'C:\Windows\System32\msiexec.exe'
        CommandLine|contains: ' sl '
    condition: 1 of selection_* and not 1 of filter_*
falsepositives:
    - Admin activity
    - Scripts and administrative tools used in the monitored environment
    - Maintenance activity
level: high
```
