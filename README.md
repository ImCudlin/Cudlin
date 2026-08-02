## Win32PrioritySeparation
- Controls thread scheduling, foreground priority boosts, and CPU quantum behavior.
- Affects application responsiveness and MMCSS task scheduling.
```batch
Reg.exe add "HKLM\SYSTEM\CurrentControlSet\Control\PriorityControl" /v "Win32PrioritySeparation" /t REG_DWORD /d "38" /f
```



## MMCSS
- NoLazyMode:
- It is a power saving feature however setting this to 1 it will Disable IdleDetection, and will make all your processes run at all times hence more CPU cycles.
```batch
Reg.exe add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile" /v "NoLazyMode" /t REG_DWORD /d "0" /f
```
<img width="1389" height="468" alt="image" src="https://github.com/user-attachments/assets/35dc2f67-7f82-4695-95bc-2c0dccd2506b" />

- LazyModeTimeout:
- It controls how quickly the scheduler enters idle mode however setting this to 25000 prevents premature throttling hence the lowest latency and best stability observed in NVIDIA benchmarks.
```batch
Reg.exe add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile" /v "LazyModeTimeout" /t REG_DWORD /d "25000" /f
```
<img width="1398" height="440" alt="image" src="https://github.com/user-attachments/assets/4e428560-f185-44f0-939d-740708753e7d" />

## NetworkThrottlingIndex
- Controls the network packet processing rate throttling applied by MMCSS. Default and recommended value is `10` — disabling throttling (`0xFFFFFFFF`) may increase interrupt overhead instead of improving performance.
```batch
Reg.exe add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile" /v "NetworkThrottlingIndex" /t REG_DWORD /d "10" /f
```
<img width="602" height="481" alt="image" src="https://github.com/user-attachments/assets/7f9a6756-5433-41c1-b489-4c12e959f71a" />

- SystemResponsiveness:
- Set system responsiveness to 10%:
- Allocates less CPU resources to tasks that request it such as browsers, so that other applications will not be impacted as much.
https://learn.microsoft.com/en-us/windows/win32/procthread/multimedia-class-scheduler-service#registry-settings
```batch
Reg.exe add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile" /v "SystemResponsiveness" /t REG_DWORD /d "10" /f
```

- Games:
- Set the Storage I/O (SFIO) priority for the Games multimedia task to High.
- This gives game-related multimedia tasks a higher storage I/O priority when scheduled.
```batch
Reg.exe add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile\Tasks\Games" /v "SFIO Priority" /t REG_SZ /d "High" /f
```



## Disable SvcHostSplit

- Forces Windows to keep services grouped into shared svchost.exe processes instead of splitting each into its own process. Reduces memory footprint and process count at the cost of per-service fault/security isolation. Xbox/Xbl-related services are explicitly excluded and left split, since forcing them into shared hosts is known to cause Xbox Live/Game Bar reliability issues.

**1. Raise the global split threshold** (tricks Windows into always treating the system as low-RAM):

```powershell
Powershell -Command "Reg.exe add 'HKLM\SYSTEM\CurrentControlSet\Control' /v SvcHostSplitThresholdInKB /t REG_DWORD /d 4294967295 /f"
```

**2. Force-disable splitting per service** (excludes Xbox/Xbl services):

```powershell
powershell -Command "Get-ChildItem 'HKLM:\SYSTEM\CurrentControlSet\Services' | ForEach-Object { if ($_.Name -match 'Xbl|Xbox') { Remove-ItemProperty -Path $_.PSPath -Name 'SvcHostSplitDisable' -ErrorAction SilentlyContinue } else { if ($null -ne (Get-ItemProperty -Path $_.PSPath -ErrorAction SilentlyContinue).Start) { New-ItemProperty -Path $_.PSPath -Name 'SvcHostSplitDisable' -PropertyType DWord -Value 1 -Force -ErrorAction SilentlyContinue | Out-Null } } }"
```

## Disable PowerSavings For All Devices
- Disables Windows power-management "allow this device to wake the computer" and "allow the computer to turn off this device to save power" settings across HID and USB input devices. Prevents devices (mice, keyboards, controllers) from being power-throttled or put to sleep, which can otherwise introduce input lag or wake-up delay.

```batch
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

echo Press any key to Skip...
pause >nul
exit /b 0
```

## Disable Unused/Software Devices

Disables a list of typically-idle system devices (radio, legacy PCI/ACPI helper devices, unused PCI bridges) via Device Manager to reduce enumerated device count. Skips PCI bridges that still have child devices attached, and re-scans devices at the end.

```batch
@echo off
setlocal EnableDelayedExpansion

:: Disable Software Radio Device
:: pnputil /disable-device "SWD\RADIO\{3DB5895D-CC28-44B3-AD3D-6F01A782B8D2}" >nul 2>&1 && echo [+] Disable Software Radio Device.

powershell -Command "$d='AMD PSP','AMD SMBus','Base System Device','Composite Bus Enumerator','Direct memory access controller','High precision event timer','Intel Management Engine','Intel SMBus','Legacy device','Microsoft Kernel Debug Network Adapter','Motherboard resources','Numeric Data Processor','PCI Data Acquisition and Signal Processing Controller','PCI Encryption/Decryption Controller','PCI Memory Controller','PCI Simple Communications Controller','PCI standard RAM Controller','SM Bus Controller','System CMOS/real time clock','System Speaker','System Timer'; Get-PnpDevice | Where-Object {$d -contains $_.FriendlyName} | ForEach-Object {Write-Host 'Disabling' $_.FriendlyName; Disable-PnpDevice -InstanceId $_.InstanceId -Confirm:$false -ErrorAction SilentlyContinue}"

:: Disable Unused PCI Bridge Devices
set "T=%TEMP%\pci%RANDOM%.txt"
pnputil /enum-devices > "%T%" 2>nul
set "id=" & set "p=0" & set "b=0"

for /f "usebackq tokens=1,* delims=:" %%A in ("%T%") do (
    set "k=%%A" & set "v=%%B" & call :trim
    if /i "!k!"=="Instance ID" (
        if "!b!"=="1" set "_id=!id!" & call :D
        set "id=!v!" & set "p=0" & set "b=0"
        set "_=!v:~0,4!" & if /i "!_!"=="PCI\" set "p=1"
    )
    if /i "!k!"=="Device Description" if "!p!"=="1" (
        echo !v! | findstr /i /c:"bridge" /c:"root port" >nul 2>&1 && set "b=1"
    )
)
if "!b!"=="1" set "_id=!id!" & call :D
del "%T%" >nul 2>&1
pnputil /scan-devices >nul 2>&1
goto :next

:trim
if "!v:~0,1!"==" " set "v=!v:~1!" & goto :trim
goto :eof

:D
pnputil /enum-devices /instanceid "!_id!" /relations 2>nul | findstr /i "Child" >nul 2>&1 && (
    echo [~] Skip: !_id! & goto :eof
)
pnputil /disable-device "!_id!" >nul 2>&1 && echo [+] Done: !_id! || echo [-] Fail: !_id!
goto :eof

:next
echo press any key to Skip...
pause >nul
```

