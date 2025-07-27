参考[🔗tvm-gdb-commands](https://github.com/anirudhsundar/tvm-gdb-commands)

用`lldb`调试时，需要安装`CodeLLDB`和`LLDB DAP`这两个插件，`LLDB DAP`可以在`VSCode Debug console`中输入`lldb`命令进行调试。



`lldb`不稳定，会出现奇怪的bug，换用gdb

可以直接用gdb调试python解释器，在TVM后端CPP打断点，跳转到TVM后端

```shell
$ gdb --args python test.py
$ b task_scheduler.cc:160
$ start
$ c
```

或者在VSCode中配置：

VSCode debug console中执行命令，例如`-exec call tvm::Dump(obj)`

```json
{
    "version": "0.2.0",
    "configurations": [
        /* 
         * Configuration 1: Hybrid Python + C++ Debugging for TVM.
         * Purpose: Debug both Python frontend and C++ backend simultaneously.
         * Mechanism: Launches Python script while attaching to C++ process.
         * Use: This is a hybrid debugging approach, where users should launch
         *      the Python script first, and then the C++ process is automatically
         *      attached to.
         * Note:
         *   - Requires `vadimcn.vscode-lldb` (CodeLLDB) extension for C++ debugging.
         *   - The following 1.1 and 1.2 configurations are subordinate to this configuration.
         */
        {
            "name": "Full Auto Debug (Python+C++)",
            "type": "pythoncpp",
            "request": "launch",
            "pythonLaunchName": "Python: TVM Frontend",
            "cppAttachName": "C++: TVM Backend",
        },
        /*
         * Configuration 1.1: Python Frontend Debugger.
         * Purpose: Debug Python components of TVM.
         */
        {
            "name": "Python: TVM Frontend",
            "type": "debugpy",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal",
            "justMyCode": false,
            "purpose": [
                "debug-in-terminal"
            ],
            "args": [
                "--opt-level=0"
            ],
            "env": {
                "PYTHONPATH": "/home/pengbenkang/github/tvm/python:${env:PYTHONPATH}",
                "TVM_NUM_THREADS": "1",
                "TVM_BACKTRACE": "1",
            },
            "gevent": false
        },
        /*
         * Configuration 1.2: C++ Backend Attach Debugger.
         * Purpose: Debug TVM's C++ runtime.
         * Note:
         *   - Customize `"program"` entry to your venv python executable path.
         */
        {
            // "name": "C++: TVM Backend",
            // "type": "lldb",
            // "request": "attach",
            // // Customize this to your venv python executable path
            // "program": "/home/yangjianchao/miniconda3/envs/tvm-build-venv/bin/python3",
            // "pid": "${command:pickProcess}",
            "name": "C++: TVM Backend",
            "type": "cppdbg",
            "request": "attach",
            "program": "/home/pengbenkang/miniconda3/envs/tvm-build-venv/bin/python3",
            "processId": "${command:pickProcess}",
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        },
        /*
         * Configuration 2: Pure C++ Debugging.
         * Purpose: Debug standalone C++ executables.
         * Use: When testing C++ components without Python
         * Note:
         *   - Customize `"program"` entry to your C++ executable path.
         */
        {
            "name": "Pure C++ Debug",
            "type": "lldb",
            "request": "launch",
            // Customize this to your executable path
            "program": "${workspaceFolder}/oot-tvm-example-project/build/oottvm_test",
            "args": [],
            "cwd": "${workspaceFolder}/oot-tvm-example-project",
            "console": "integratedTerminal",
            "env": {
                "EXAMPLEKEY": "EXAMPLEVALUE"
            },
        },
    ],
}
```

