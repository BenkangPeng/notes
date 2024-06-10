### testbench中的阻塞赋值与非阻塞赋值

下面这两端代码中，`en`在`testbench`中有两种赋值方式：`en = 1;`和`en <= 1;`

* `en=1;`会让`en`在时钟上升沿前立刻变为1.因此`valid`会在`(10+time1)*clock_period`时上升为1.
* `en<=1`是非阻塞赋值，在时钟上升沿过后，`en`才会变成1(在`(10+time1+time2)`时上升为1).因此`valid`在`(10+time1+time2+1)`时上升为1

> =是用于阻塞赋值（Blocking Assignment）。在仿真中，阻塞赋值会立即更新信号的值，并且在执行下一条语句之前完成这个操作。
>
> <=是用于非阻塞赋值（Non-blocking Assignment）。在仿真中，非阻塞赋值会在当前仿真时间单位结束时同步更新所有的值。这意味着在当前时间点的所有非阻塞赋值语句将被评估，但赋值直到所有当前时刻的仿真事件都处理完毕后才会发生。

```verilog
always_ff @(posedge clk or negedge rstn)begin
    if(~rstn)begin
       valid <= 0; 
    end
    else if(en) begin
        valid <= 1;
    end
    else begin
       valid <= 0; 
    end
    
end
```

```verilog
initial begin
   	rstn = 0;
    en = 0;
    #(10*clock_period)
    rstn = 1;
    
    #(`time1 * clock_period)
    en = 1;
    
    #(`time2 * clock_period)
    en <= 1;
    
    #(10 * clock_period)
    $finish;
end
```













