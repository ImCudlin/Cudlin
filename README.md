## Win32PrioritySeparation
- Controls thread scheduling, foreground priority boosts, and CPU quantum behavior. Affects application responsiveness and MMCSS task scheduling.
```cmd
Reg.exe add "HKLM\SYSTEM\CurrentControlSet\Control\PriorityControl" /v "Win32PrioritySeparation" /t REG_DWORD /d "38" /f
```
