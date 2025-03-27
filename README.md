Adding an agent to wazuh server is very easy and there is no better guide for than tha Wazuh documentation.

What is important to note, that windows powersell and wazuh are not friends and they don't get along, so wazuh woud't inform us about suspicious powershell processes, which is a key for endpoint monitoring. Luckily we can do a couple tweaks to help them find common language. 

go to gpedit.msc> administrative templtes > windows components and then Windows Powershell
Turn on Module logging > Enabled > show > type "*" > ok
Script block logging > Enabled 
Turn on Powershell Transcription > Enabled * Include invocation Headers
Event Viewer > Applciation > Windows Powershell 

now go to C:\Program Files (x86)\ossec-agent/win32ui > view > view config 

add: 
``
<localfile>
<location>Microsoft-Windows-PowerShell/Operational</location>
<log_format>eventchannel</log_format>
</localfile>
``

And restart agent 

Now the last step is to add rules in wazuh for powershell 

