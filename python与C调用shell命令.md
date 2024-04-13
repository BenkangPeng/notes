[参考链接](https://mp.weixin.qq.com/s/ahXrzTlIJcQGW_ONPvRgJQ)
`C++`调用`shell`命令:

```cpp
char* command = malloc(sizeof(char)*100);
    snprintf(command , 100 , "sed -i '13c `define M_c %d' ../sim/testbench/mau_sau_tb_fp64.sv" , max_row);
    system(command);
    free(command);
```

`python`调用`shell`命令

```python
os.system("echo 'hello world'")
```


