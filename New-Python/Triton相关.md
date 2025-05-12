### gpu切换报错

使用`torch.cuda.set_device(1)`设置gpu
```python
import torch
import triton
import triton.language as tl
torch.cuda.set_device(1)
DEVICE = 'cuda:1' if torch.cuda.is_available() else 'cpu'

```

如果直接使用`DEVICE = 'cuda:1' if torch.cuda.is_available() else 'cpu'`
可能出现报错：
```shell
RuntimeError: Triton Error [CUDA]: device kernel image is invalid
```



### sub-group分块矩阵乘优化分析

```shell
ncu  --set detailed -o matmul python PerformanceTest.py

```
