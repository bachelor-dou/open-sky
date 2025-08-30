### llama.cpp

```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "test-backend-ops",
            "type": "cppdbg",
            "request": "launch",
            "cwd": "${workspaceFolder}",
            "program": "${workspaceFolder}/build/bin/test-backend-ops",
            "args": [
                "test",
                "-b",
                "CANN0",
                "-o",
                "DUP"
            ],
            "MIMode": "gdb",
            "targetArchitecture": "arm64",
            "stopAtEntry": false,
            "externalConsole": false,
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        },
        {
            "name": "llama-bench",
            "type": "cppdbg",
            "request": "launch",
            "cwd": "${workspaceFolder}",
            "program": "${workspaceFolder}/build/bin/llama-bench",
            "args": [
                "-m",
                //"/home/dou/models/qwen2.5:0.5b-instruct-fp16",
                "/home/dou/models/Hermes-2-Pro-Llama-3-8B-f16.gguf",
                "-p",
                "Building a website can be done in 10 simple steps:",
                "-ngl",
                "32",
                "-o",
                "json"
            ],
            "MIMode": "gdb",
            "targetArchitecture": "arm64",
            "stopAtEntry": false,
            "externalConsole": false,
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}

```

