# Win32PrioritySeparation
- Controls thread scheduling, foreground priority boosts, and CPU quantum behavior.
- Affects application responsiveness and MMCSS task scheduling.
```batch
Reg.exe add "HKLM\SYSTEM\CurrentControlSet\Control\PriorityControl" /v "Win32PrioritySeparation" /t REG_DWORD /d "38" /f
```

# MMCSS
- It is a power saving feature however setting this to 1 it will Disable IdleDetection, and will make all your processes run at all times hence more CPU cycles.
```batch
Reg.exe add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile" /v "NoLazyMode" /t REG_DWORD /d "0" /f
```
