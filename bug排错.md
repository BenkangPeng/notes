### docker容器中无法使用gdb

显示报错：

> warning: Error disabling address space randomization: Operation not permitted 
>
> warning: Could not trace the inferior process. 
>
> warning: ptrace: Operation not permitted During startup program exited with code 127.

需要在docker启动时加上`--privileged`

```shell
docker run  --privileged --name pbk-llvm -it allo:llvm-18.x-py3.12-v1.0.1 /bin/bash
```

