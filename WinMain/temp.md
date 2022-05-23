# WinMain

# **INDEX**

**1. [Commandline 인자 받기](#Commandline-인자-받기)**


# **Commandline 인자 받기**

GetCommandLine함수를 통해서 인자를 파싱할 수 있다.

```c++
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, PSTR szCmdLine, int iCmdShow){
    TCHAR cmdline[1024];
    TCHAR* argv[3];
    int argc = 0;


    _tcscpy(cmdline, GetCommandLine());
    argv[argc] = _tcstok(cmdline, TEXT(" \t"));
    while (argv[argc] != 0)
        argv[++argc] = _tcstok(0, TEXT(" \t"));
    return 0;
}
```