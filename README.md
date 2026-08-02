# Win32PrioritySeparation
- Controls thread scheduling, foreground priority boosts, and CPU quantum behavior.
- Affects application responsiveness and MMCSS task scheduling.
```batch
Reg.exe add "HKLM\SYSTEM\CurrentControlSet\Control\PriorityControl" /v "Win32PrioritySeparation" /t REG_DWORD /d "38" /f
```

# MMCSS
- NoLazyMode:
- It is a power saving feature however setting this to 1 it will Disable IdleDetection, and will make all your processes run at all times hence more CPU cycles.
```batch
Reg.exe add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile" /v "NoLazyMode" /t REG_DWORD /d "0" /f
```
- LazyModeTimeout:
- It controls how quickly the scheduler enters idle mode however setting this to 25000 prevents premature throttling hence the lowest latency and best stability observed in NVIDIA benchmarks.
```batch
Reg.exe add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile" /v "LazyModeTimeout" /t REG_DWORD /d "25000" /f
```
- NetworkThrottlingIndex:
- Network Packet Processing Rate.
- :: 10 = Recommended. Disabling throttling may increase interrupt overhead.
```batch
Reg.exe add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile" /v "NetworkThrottlingIndex" /t REG_DWORD /d "10" /f
```
