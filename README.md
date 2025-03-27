Adding an agent to the Wazuh server is straightforward, and there's no better guide for it than the official Wazuh documentation 📚.

However, it's important to note that Windows PowerShell and Wazuh don’t get along 🤝❌. As a result, Wazuh may not notify us about suspicious PowerShell processes, which is a critical aspect of endpoint monitoring 🔍💻. Fortunately, with a few tweaks, we can help them find common ground and improve monitoring ⚙️

go to gpedit.msc> administrative templtes > windows components and then Windows Powershell
Turn on Module logging > Enabled > show > type "*" > ok
Script block logging > Enabled 
Turn on Powershell Transcription > Enabled * Include invocation Headers
Event Viewer > Applciation > Windows Powershell 


Alternatively, run a command: 


function Enable-PSLogging {
    # Define registry paths for ScriptBlockLogging and ModuleLogging
    $scriptBlockPath = 'HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging'
    $moduleLoggingPath = 'HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging'
    
    # Enable Script Block Logging
    if (-not (Test-Path $scriptBlockPath)) {
        $null = New-Item $scriptBlockPath -Force
    }
    Set-ItemProperty -Path $scriptBlockPath -Name EnableScriptBlockLogging -Value 1

    # Enable Module Logging
    if (-not (Test-Path $moduleLoggingPath)) {
        $null = New-Item $moduleLoggingPath -Force
    }
    Set-ItemProperty -Path $moduleLoggingPath -Name EnableModuleLogging -Value 1
    
    # Specify modules to log - set to all (*) for comprehensive logging
    $moduleNames = @('*')  # To specify individual modules, replace * with module names in the array
    New-ItemProperty -Path $moduleLoggingPath -Name ModuleNames -PropertyType MultiString -Value $moduleNames -Force

    Write-Output "Script Block Logging and Module Logging have been enabled."
}

Enable-PSLogging





Open command prompt and run: gpupdate /force


now go to C:\Program Files (x86)\ossec-agent/win32ui > view > view config 

add: 
``
<localfile>
<location>Microsoft-Windows-PowerShell/Operational</location>
<log_format>eventchannel</log_format>
</localfile>
``

And restart agent 

Now the last step is to add rules in wazuh for powershell and restart manager. 

Now, everything should be work

