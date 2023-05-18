# bios_editor
Powershell script to turn off secureboot and switch from RAID to AHCI



# Check if running with administrator privileges
if (-not ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")) {
    Write-Host "Please run this script with administrative privileges."
    Exit
}

# Disable Secure Boot
Write-Host "Disabling Secure Boot..."

# Replace 'BIOSProvider' with the appropriate value for your hardware manufacturer
$BIOSProvider = "AMI"
$SecureBootSetting = 0x1 # Replace with the appropriate value for disabling Secure Boot

# Execute the command to disable Secure Boot
Set-WmiInstance -Namespace "root\wmi" -Class "Wmi_SecureBootSetting" -Arguments @{Manufacturer=$BIOSProvider;SecureBoot=$SecureBootSetting} | Out-Null

Write-Host "Secure Boot has been disabled successfully."

# Switch Storage Mode from RAID to AHCI
Write-Host "Switching Storage Mode from RAID to AHCI..."

# Replace 'PathToRegFile' with the path to the registry file to change the storage mode
$PathToRegFile = "C:\temp\storagemode.reg"

# Define the content of the registry file
$RegistryContent = @"
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\storahci]
"Start"=dword:00000000

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\iaStorV]
"Start"=dword:00000004
"@

# Write the content to the registry file
$RegistryContent | Out-File -FilePath $PathToRegFile -Force -Encoding ASCII

# Import the registry file to switch the storage mode
reg import $PathToRegFile

# Clean up the registry file
Remove-Item -Path $PathToRegFile -Force

Write-Host "Storage Mode has been switched to AHCI successfully."
