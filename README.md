## Win32PrioritySeparation
- Controls thread scheduling, foreground priority boosts, and CPU quantum behavior.
- Affects application responsiveness and MMCSS task scheduling.
```cmd
Reg.exe add "HKLM\SYSTEM\CurrentControlSet\Control\PriorityControl" /v "Win32PrioritySeparation" /t REG_DWORD /d "38" /f
```

## MMCSS
1. NoLazyMode:
- It is a power saving feature however setting this to 1 it will Disable IdleDetection, and will make all your processes run at all times hence more CPU cycles.
```cmd
Reg.exe add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile" /v "NoLazyMode" /t REG_DWORD /d "0" /f
```
<img width="1389" height="468" alt="image" src="https://github.com/user-attachments/assets/35dc2f67-7f82-4695-95bc-2c0dccd2506b" />

2. LazyModeTimeout:
- It controls how quickly the scheduler enters idle mode however setting this to 25000 prevents premature throttling hence the lowest latency and best stability observed in NVIDIA benchmarks.
```cmd
Reg.exe add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile" /v "LazyModeTimeout" /t REG_DWORD /d "25000" /f
```
<img width="1398" height="440" alt="image" src="https://github.com/user-attachments/assets/4e428560-f185-44f0-939d-740708753e7d" />

3. NetworkThrottlingIndex:
- Controls the network packet processing rate throttling applied by MMCSS. Default and recommended value is `10` — disabling throttling (`0xFFFFFFFF`) may increase interrupt overhead instead of improving performance.
```cmd
Reg.exe add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile" /v "NetworkThrottlingIndex" /t REG_DWORD /d "10" /f
```
<img width="602" height="481" alt="image" src="https://github.com/user-attachments/assets/7f9a6756-5433-41c1-b489-4c12e959f71a" />

4. SystemResponsiveness:
- Set system responsiveness to 10%:
- Allocates less CPU resources to tasks that request it such as browsers, so that other applications will not be impacted as much.
https://learn.microsoft.com/en-us/windows/win32/procthread/multimedia-class-scheduler-service#registry-settings
```cmd
Reg.exe add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile" /v "SystemResponsiveness" /t REG_DWORD /d "10" /f
```

5. Games:
- Set the Storage I/O (SFIO) priority for the Games multimedia task to High.
- This gives game-related multimedia tasks a higher storage I/O priority when scheduled.
```cmd
Reg.exe add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile\Tasks\Games" /v "SFIO Priority" /t REG_SZ /d "High" /f
```

## Disable SvcHostSplit
Forces Windows to keep compatible services grouped into shared svchost.exe processes instead of splitting each service into its own process by setting the ```SvcHostSplitDisable``` registry value. This reduces overall memory usage and the number of running processes at the cost of per-service fault and security isolation. Xbox/Xbl-related services are explicitly excluded and left to use Windows' default behavior, as forcing them into shared service hosts is known to cause Xbox Live and Game Bar reliability issues.

```powershell
Powershell -Command "if (-not ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) { Write-Host 'ERROR: Must run as Administrator!' -ForegroundColor Red; exit }; Get-ChildItem 'HKLM:\SYSTEM\CurrentControlSet\Services' | ForEach-Object { $svc = $_.PSChildName; if ($svc -match 'Xbl|Xbox') { Remove-ItemProperty -Path $_.PSPath -Name 'SvcHostSplitDisable' -ErrorAction SilentlyContinue; Write-Host \"[~] Excluded: $svc\" -ForegroundColor Yellow; return }; if ($null -ne (Get-ItemProperty -Path $_.PSPath -Name 'SvcHostSplitDisable' -ErrorAction SilentlyContinue)) { Write-Host \"[=] Already set: $svc\" -ForegroundColor DarkGray; return }; try { New-ItemProperty -Path $_.PSPath -Name 'SvcHostSplitDisable' -PropertyType DWord -Value 1 -Force -ErrorAction Stop | Out-Null; Write-Host \"[+] Disabled SvcHost splitting: $svc\" -ForegroundColor Green } catch { Write-Host \"[-] Skipped (protected): $svc\" -ForegroundColor Red } }"
```

## Disable PowerSavings For All Devices
- Disables Windows power-management "allow this device to wake the computer" and "allow the computer to turn off this device to save power" settings across HID and USB input devices. Prevents devices (mice, keyboards, controllers) from being power-throttled or put to sleep, which can otherwise introduce input lag or wake-up delay.

```cmd
@echo off
color 07
title Disable Power Savings For All Devices

:: Disable HID Power Savings Devices
powershell -Command Write-Host "[+] Disable HID PowerSavings Devices:" -foregroundcolor green
for /f "delims=" %%D in ('powercfg -devicequery wake_programmable') do (
    echo Disabling wake for: %%D
    powercfg -devicedisablewake "%%D"
)

:: Disable Power Savings USB Input Devices
powershell -Command Write-Host "[+] Disable Power Savings USB Input Devices:" -foregroundcolor green
powershell -Command "Get-CimInstance -Query 'SELECT * FROM MSPower_DeviceEnable' -Namespace 'root\WMI' | ForEach-Object { $id = $_.InstanceName; if ($id.EndsWith('_0')) { $id = $id.Substring(0, $id.Length - 2) }; $dev = Get-PnpDevice -InstanceId $id -ErrorAction SilentlyContinue; $name = if ($dev.FriendlyName) { $dev.FriendlyName } else { $_.InstanceName.Split('\\')[-1] }; Write-Host \"Disabling Power Savings For: $name\"; Set-CimInstance -CimInstance $_ -Property @{Enable=$false} }"

pause >nul
exit /b 0
```

