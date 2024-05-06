2023-12-21 20:53

需要改的地方

- [ ] DAG、DRU，MJ师兄的方案不需要打斜，可以用decoder代替DRU，encode加在DAG之前？
- [ ] SAG前加一个encode , 之后加一个decode ?  不对！MJ师兄B矩阵没有编解码。
- [ ] 把软件部分改一下吧，硬件还不敢改:joy:
- [ ] 继续读SIGMA吧，找到一个模拟器可以模拟脉动阵列：SCALE sim

明天干些啥：

:date:2023-12-22 20:23

* `sau_single_test.sh`中注释`11-13`行，启用`15-17` , (我将16改成4了，原来是4)。
* `macmatrix.py`中`60-70`行的`a[i][j]`赋值不要用随机数了，换成自定义的数看效果

:date:2023-12-24 22:06

* 珉琎师兄代码里`test_suitSparse_mm.cpp`中没有`random_spMatrix.h`,导致找不到函数`generateSparseMatrix`
* 带Latency的`encode45`merge进师兄的代码了。`encode135`还没merge
* 师兄说SA前可以搞条专门的旁路

:date: 2024-01-03 22:36

* 目前CPU代码跑的是一个``双数32位``的demo , 即：64位的a ， 里面存了两个32位的数 ； 同时，64位的b里面也存了两个32位的数，且两个数相同。在PE的MAC中，将a中的两个数分别拉出，与b中的两个数分别相乘。同样的，C中也存了两个32位的数。
  
  例如 ： `a = 64'h40e0_0000_4040_0000` , `b = 64'h4150_0000_4150_0000` , `c = 256'h40A0_0000_3F80_0000` , 十进制表示是`a = {7 , 3} , b = {13 , 13} , c = {0 , 0 , 0 , 0 , 0 , 0 , 5 , 1}` , 做乘加：7×13+5 = 96 ， 3×13+1 = 40

* 我们的稀疏是做到哪种精度上？`双数32`or `单数64`or `单数32` ？

单数64最好改，单数32把高位(或低位)抹零即可(去看数据生成预处理部分) ， 双数32匹配idx要重复一次

* 稀疏模式与稠密模式的切换，拉出一根信号线`flag`即可

:date:2024-01-08 16:51

MAC有这样的模式吗：例如输入`VMACSrc1 = 64'h0000_0002_4080_0000` (高32位表示整数2，该数的行号为2，低三十二位表示浮点数4.0)，`VMACSrc2 = 64'h0000_0001_4150_0000` , `VMACSrc3 = 0` , 乘加结果 `VMAC_Dst = 255'h0000_000`

团支部成员积极向党组织靠拢，并推荐、引导优秀团员递交入党申请书。在团支部的号召下，已有崔璞钰、曹长天等17名正式党员、预备党员。在本科生微电子第二党支部，本团支部成员表现先锋模范作用：胡康威同志担任党支部纪检委员，杨璐语同志担任党支部宣传委员，在职期间表现优秀、工作认真负责，获得了广泛认同。
团支部成员学习努力，杨骏豪、孙科等多名团员保研进入研究生阶段深造学习，考研的同学预计将取得优秀的成绩。团支部团员在多项科技竞赛中取得优秀成绩：许达城同志取得全国集成电路创新创业大赛全国决赛一等奖。
团支部积极开展团课交流，认真学习习近平新时代中国特色社会主义思想，积极展开团课学习交流，定期展开组织生活会，全员参与团课学习，团员参会率95%以上。
在社会服务、志愿服务上，团支部同学积极参与：崔璞钰同志连续多年担任年级助理，为同学们认真服务；曾如利同志担任团总支书，负责学院团委事务；
总体上，经团支部内部讨论自评，我们认为微电子2004班团支部在理论学习、科技创新、社会服务等多方面起到了模范作用，自评为五星级团支部。

:date:2024-01-11 17:00

c_buffer中sau_wrt_valid使得wrt_data0_1只取了sau_wrt_data的一部分

VCS如何搜索，如何复制粘贴

`X`显示verdi文本中变量的数值

gold里的

改了testmode.py中的line125~127

$n_{n0}=n_ie^{\frac{E_{Fn}-E_i}{k_0T}}$ 

$W = $

:date: 2024-01-15

完成单点测试（4x4)

:date: 2024-01-18

B分块出现错误，定位bug: ramb(之前注释过的内容是否影响分块)  -> data_after_dec3、2出现错误，第7列8列值被4，3列替代。

继续追，发现data_before_dec3、2也错误->data_out3、2错误->data_out3、2错误->追到mau_ramb.sv中mau_ram_model的例化  ：

例化了4个模块rama_0,rama_1,rama_2,rama_3  ,     对于rama_0,rama_1,输入输出均为正确，rama_2,rama_3输入data_in2,data_in3正确,index2,index3发生错误 ， 去理解index的意思，index3,2由addr3,2驱动  ， addr3,2出错在赋值时:

**b矩阵分块索引下一个block时，因为b是分成4×4的小块，因此地址偏移add_offset=4 。因为稀疏是用的FP32，SIMD模式，SIMD会把B复制一份，add_offset=2.** 把mau_sau_sag.svline213行修改

:date:01-19

<del>`mau_c_buffer.sv`中line685`sau_wrt_valid`有一段没拉高，`sau_wrt_data`有一部分没传给`wrt_data0_1` , c_buffer_adu_data中缺少的就是这一部分</del>

`wrt_data0_1`中数据是正确的

mau_c_buffer.sv  line937  :`rd_data0_0`第一拍只有一个数(1,1,c0e22cb6), `rd_data0_0`有部分数据未传给`c_buffer_adu_data_t`

感觉仍然是模式的问题,去查data_out00的来源

去查data_out00\01\02\03给rd_data0_0的驱动

`acu_c_buffer_c_row` = A的行数，`acu_c_buffer_c_col` = A 的列数

`acu_c_buffer_a_blocks_row` = 1

python实现高精度32位浮点数加法：输入两个长度为8的字符串，表示16进制下的32位浮点数，输出一个长度为8的字符串，表示两个浮点数相加的结果的16进制表示。

`data_y_t.dat`文件中有很多行，我向你解释如下：每一行表示索引和数，例如`00010001c08f014a`表示索引为`00010001`,数值是`c08f014a`(32位浮点数-4.468)。现在我要将`data_y_t.dat`中相同索引的行合并，合并后的行内容为索引+值，索引为相同的索引，值为浮点数相加后的值。例如第一行`00010001c08f014a`与第九行`00010001c21be129`索引相同，进行合并，合并后这一行变成`00010001C22DC083` , 其中`00010001`是两行相同的索引。`C22DC083`是两行的值相加的结果。`data_y_t.dat`中所有相同索引的行合并后，将合并后的行写回`data_y_t.dat`

:date:2024-01-12

==写成文章==

今天在跑重行师兄的python数据脚本的时候遇到报错：

> **ModuleNotFoundError: 找不到指定的模块**

> **ImportError: No module named 'xxx'**

到网上查都是添加`sys.path` :  

在文件前添加：

```python
import sys , os
sys.path.append(os.path.dirname(os.path.abspath(__file__)))
```

把当前文件所在的文件夹添加进python import时搜寻的路径。

或者在文件夹中添加一个·`__init__.py`文件加上以上内容，把当前路径添加进import搜寻module的范围

结果还是不行：

师兄一上来就查python版本，查出来回退到Python2.7了，说明conda失效了，重启一下anaconda就行了

```bash
$ conda deactivate
$ conda activate
```

==为什么我们的脉动只能算mkn是四的倍数==

==要探索的地方：==

* a,b有一个为0就停掉MAC，节省功耗
* 数据后处理能不能用加法树做，与软件处理耗时做对比
* 

==mau_sau_dag.sv中的dag_c_buffer_send_dr_en信号有问题==

今天改mau_sau_dag.sv就改了3个地方：Line296 , Line299-300 , Line443

1.成熟的模块尽量不做修改, 后果往往是走了弯路后git版本回滚L

●

2.改之前分析清楚原代码逻辑，不熟悉代码逻辑使得我进展较慢

●

3.一定记得备份……

●

4.配套信号要看全，data,valid,mask信号要一起看->

:date: 01-28

- [x] mkn只能是4的整数吗
- [x] 重写tb改成读文件

:date:01-29

sau_single_test.sh : line 15 , 37

gemm_factory.py:line51,69,73

float32matrix.py line14,15

macmatrix.py line 12 ,15,192

fp32_mul_fp32 .py  line12,20

testmode.py:line29,50

改绝对路径：sau_single_test.sh  L25,  vcs_local.sh  , verdi_local.sh

-3.07560586929321340c0000

6

2.924394130706787

:date:02-01

```
benkangpeng@DESKTOP-FR84659 MINGW64 /d/Desktop/sparse/sau_out/ai_4x4_env/data (master)
$ gcc gemm_factory.c -o gemm_factory.out && ./gemm_factory.out 8 4 4 0.5 8 0
   80.03     0.00     0.00     0.00 
  -47.67     0.00     0.00     0.00
   69.85     0.00     0.00    87.69
    0.00     0.00    63.17   -69.04
   83.66    72.52   -66.76   -75.22
    0.00    -0.21    71.01     0.00
    0.00   -77.31    39.64     0.00
    0.00     0.00    97.79   -97.75
       1        0        4        4
       3        0        6        3
       5        5        5        5
   80.03     0.00    63.17   -69.04
   69.85    -0.21    71.01    87.69
   83.66    72.52   -66.76   -75.22
```

====

```
# Character <Bot 人设>
你是一位计算机科学领域专家，精通C/C++、Python、Systemverilog等各种编程语言，精通计算机体系结构、人工智能、软件工程、AI加速器、芯片设计等领域知识，你能准确、详细、耐心地解答各种问题。

## Skills <Bot 的功能>
### Skill 1: 代码实现
1. 当用户提供一个代码实现的功能需求时，能准确无误地给出代码实现，并对代码进行解释。
2. 如果用户提供源代码并询问代码细节时，能准确无误地给出代码解释。

### Skill 2: 问题解答
1. 当用户询问计算机科学专业知识时，优先从知识库中寻找答案，并给出准确的回答。
2. 通过查询知识库、数据库、搜索网络来给出准确的回答。

## Constraints <Bot 约束>
- 准确回答问题，不编造、虚构回答。
- 所输出的内容按照markdown格式输出。
- 在使用特定编程语言解决问题时，必须解释所使用的逻辑和方法，不能仅仅给出代码。
```

:date:20240221

**稠密模式下**

```
****************************
M_32    K_32    N_16    dense mode 
load data cost 5155 ns
compute cost 10650 ns
store data cost 1390 ns
sim totally cost 17195 ns
****************************

****************************
M_12    K_32    N_16    dense mode 
load data cost 2755 ns
compute cost 13130 ns
store data cost 590 ns
sim totally cost 16475 ns
****************************
```

```
****************************
M_64    K_32    N_16    dense mode 
load data cost 8995 ns
compute cost 20890 ns
store data cost 2670 ns
sim totally cost 32555 ns
****************************

****************************
M_32    K_32    N_16    dense mode 
load data cost 5155 ns
compute cost 10650 ns
store data cost 1390 ns
sim totally cost 17195 ns
****************************

****************************
M_16    K_32    N_16    dense mode 
load data cost 3235 ns
compute cost 14400 ns
store data cost 750 ns
sim totally cost 18385 ns
****************************
```

**稀疏**

从稠密实验结果看来，M小到32左右，计算耗时发生异常

稀疏递减组(不压缩)

```yml
****************************
M_64    K_32    N_16    Mc_64
load data cost 8995 ns
compute cost 20890 ns
store data cost 10320 ns
sim totally cost 40205 ns
****************************

****************************
M_56    K_32    N_16    Mc_56
load data cost 8035 ns
compute cost 18330 ns
store data cost 9040 ns
sim totally cost 35405 ns
****************************

****************************
M_48    K_32    N_16    Mc_48
load data cost 7075 ns
compute cost 15770 ns
store data cost 7760 ns
sim totally cost 30605 ns
****************************

****************************
M_40    K_32    N_16    Mc_40
load data cost 6115 ns
compute cost 13210 ns
store data cost 6480 ns
sim totally cost 25805 ns
****************************

****************************
M_32    K_32    N_16    Mc_32
load data cost 5155 ns
compute cost 10650 ns
store data cost 5200 ns
sim totally cost 21005 ns
****************************

****************************
M_24    K_32    N_16    Mc_24
load data cost 4195 ns
compute cost 8990 ns
store data cost 3920 ns
sim totally cost 17105 ns
****************************


****************************
M_16    K_32    N_16    Mc_16
load data cost 3235 ns
compute cost 14400 ns
store data cost 2640 ns
sim totally cost 20275 ns
****************************

****************************
M_8    K_32    N_16    Mc_8
load data cost 2275 ns
compute cost 11920 ns
store data cost 1360 ns
sim totally cost 15555 ns
****************************

****************************
M_4    K_32    N_16    Mc_4
load data cost 1795 ns
compute cost 10650 ns
store data cost 720 ns
sim totally cost 13165 ns
****************************
```

发现奇怪现象：

M=16时

<img title="" src="file:///D:/Software/MarkText/images/e3f19145f0e923d1f53f4a30ef544acc99dc6f2c.png" alt="1708518158834" data-align="inline">

`dag2dr_data`中被插入了很多零,往回追踪信号：

`a_buffer`输出`a_buffer_sau_data`信号中被`0`隔开：

![1709088362436](D:\Software\Typora\Pictures\1709088362436.png)

因此之后的`dag` `dru`输出的数据也被很长的`0`隔开：

![1709088481489](D:\Software\Typora\Pictures\1709088481489.png)

![1709088528413](D:\Software\Typora\Pictures\1709088528413.png)

![1709088580897](D:\Software\MarkText\images\1e6a06d28cbed64c1c9b43f0ff399dcf37099919.png)

追信号进入了`mau_rama.sv` , 这个问题稠密模式下也存在

↑这个问题搁置，重写compare.cpp时发现①compare.py有问题，②tb中`Y_lines*2`仍然存在模糊

①`compare.py`使用不正确，`sau_single_test.sh`中`mode`和`TYPEy`只传入`compare.py`.稀疏模式应该设为`mode=0 TYPEy='float64'`

②tb中应该改成

```verilog
assign store_y_done = start_store_y & (cnt_y == (flag_sparsity?(`Y_lines-1):`Y_lines-4)) & ~store_y_done_d0 & ~store_y_done_d1 & ~store_y_done_d0 & ~store_y_done_d2;
```

```verilog
if(cnt_y == `Y_lines-1)
// if(cnt_y == `Y_lines-4)
```

部件`buffer`中存储了大量的数据，每一个数据的格式是：线宽为64位，其中低32位([31:0])表示IEEE 754标准下的32位浮点数，往上16位([47:32])表示一个整数(行号)，[63:48]表示另一个整数(列号)。例如16进制下，`64'h0008_000A_3F80_0000`表示第8行、第10列的浮点数1.0。

我需要将buffer中所有数据相加合并：若两个数据的高32位([63:32])相同，则将低32位([31:0])按照浮点数加法相加，例如buffer中有三个行号、列号相同的数:`64'h0008_000A_3F80_0000`  `64'h0008_000A_4000_0000`  `64'h0008_000A_4040_0000` , 合并成`64'h0008_000A_40C0_0000`存入存储体中。

**Testbench**

1. 考虑latency的encode
2. 图着色分列压缩
3. 数据后处理

```
APU：1. fig11中将激活矩阵分成四份放入FIFO-A中，FIFO-A#1中放PID=0的数，FIFO-A#i中放PID=i-1的数； 2. fig9中FIFO-A中的数如何通过Routing Module进入FIFO-B呢？回到fig11 , PE中的weight是固定的，例如PE#1中恒为[1 0 ; 3 0] , WSP_1为1010(weight的bitmap) ,PID=0123 , WSP_1[1],WSP_1[3]=0表示这两个位置的数为0，也就意味着PID=1,3的激活不用送进该PE(该FIFO-B)因此，我们看到FIFO-A#1(PID=0)进入了FIFO-B(其中权重有PID=0)，不进入FIFO-B#2(无权重PID=0). 而FIFO-A#3(PID=2)均进入FIFO-B#1,2 (两个FIFO-b均有PID=2)3. 
```

:date: 20240318

计算耗时在M=16激增，注意`cnt_max_tmp`  `cnt_for_update_b`  `update_b_cnt_en`信号的检查

:date:20240330

改进外积法的最后的累加，减少数据访存？

参考GROW@2023 ， 对DSTC进行评估(GROW要求图邻接矩阵)

RM-TC  fig9中 A-random mode中的32选1是个大问题

:date:20240403

初步确定是dag的bug :  dag发送给a_buffer的`dag_a_buffer_raddr` . `dag_a_buffer_rvalid`都是间断的，从而导致a_buffer送到dag的数据也是间断的

先把dag看明白吧，用一个正确的case(32x8x8)

:date: 20240409

`dag`模块中有些不懂的信号：

| subduction_t     | update_B_OK_cnt_t |
| ---------------- |:-----------------:|
| en_send_a_c      | block_gap_cnt     |
| send_new_a_c_gap | rd_gap            |
| send_rd_req_ok   |                   |
|                  |                   |
|                  |                   |

Line214这个组合逻辑不太懂

Line243数据流流入SA的拍数

* 如何通过计数更新b



:date: 20240418

显示器最佳配置：1.显示器本身的“预设模式”为暖色 2. [Iris Pro](https://www.52pojie.cn/thread-1130826-1-1.html)设置为自动、健康

:date:20240426

我将对以下C代码进行提问。该代码是将文本文件`data_y_hw.dat`中的每一行字符串读出，并输出到终端，`data_y_hw.dat`文件内容见下。该代码执行结果满足预期效果。但我在用gdb调试时(gdb调试信息如下)，发现了一些不懂的地方，特向你请教。在while循环中，第一次循环读到的line是00040001407df47c，符合预期，但第二次循环读到的line却变成了"\n\000\060\064\060\060\060\061\064\060\067df47c"，接下来的循环也出现了类似的未知的字符串，请问这是为什么？
```c
#include<stdio.h>
#include<stdlib.h>
#include<malloc.h>
#include<string.h>
int main()
{
    char* file_path = "data_y_hw.dat";
    FILE* fp = fopen(file_path , "r");

    char line[17];
    while(fgets(line , sizeof(line) , fp) != NULL){
        // line[strcspn(line,"\n")] = '\0';
        printf("%s" , line);
    } 

    fclose(fp);
    return 0;
}
```

data_y_hw.dat:

```
00040001407df47c
0001000140c5aa8c
0001000142a326d4
0004000140b81d40
0001000241d8ad03
00020002c20d4bc7
00040003c0df7eac
00040003c0f43001
00040003c26321c7
00040002c046cd70
00010004c27b2a5a
00010004418ddd46
```

gdb调试信息：

```shell
benka@MASALab MINGW64 /d/Desktop/OnWorking/cs/cpp (master)
$ gcc -g test.c -o test.out 

benka@MASALab MINGW64 /d/Desktop/OnWorking/cs/cpp (master)
$ gdb test.out 
GNU gdb (GDB) 11.2
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-w64-mingw32".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from test.out...
(gdb) start
Temporary breakpoint 1 at 0x1400014b1: file test.c, line 8.
Starting program: D:\Desktop\OnWorking\cs\cpp\test.out
[New Thread 31232.0x8b1c]

Thread 1 hit Temporary breakpoint 1, main () at test.c:8
8           char* file_path = "data_y_hw.dat";
(gdb) n
9           FILE* fp = fopen(file_path , "r");
(gdb)
12          while(fgets(line , sizeof(line) , fp) != NULL){
(gdb)
14              printf("%s" , line);
(gdb) display line
1: line = "00040001407df47c"
(gdb) n
00040001407df47c12          while(fgets(line , sizeof(line) , fp) != NULL){
1: line = "00040001407df47c"
(gdb)
14              printf("%s" , line);
1: line = "\n\000\060\064\060\060\060\061\064\060\067df47c"
(gdb)

12          while(fgets(line , sizeof(line) , fp) != NULL){
1: line = "\n\000\060\064\060\060\060\061\064\060\067df47c"
(gdb)
14              printf("%s" , line);
1: line = "0001000140c5aa8c"
(gdb)
0001000140c5aa8c12          while(fgets(line , sizeof(line) , fp) != NULL){
1: line = "0001000140c5aa8c"
(gdb)
14              printf("%s" , line);
1: line = "\n\000\060\061\060\060\060\061\064\060c5aa8c"
```





```c
M= 4;
K =4;
N =4;
row block = M;
float sparsity =0.7;
int structuredSparsity;
for(int i=0 ;i<argc; i++){
    if(i==1){sscanf(argv[1l,"SU"&M);
      else if(i=-2){sscanf(argv[2],"%u"，&K);
      else if(i==3){sscanf(argv[3],"%u",&N);
      else if(i==4){sscanf(argv[4],"%f"&sparsity);}
      else if(i==5){ 			 scanf(argv[5l,"%d",&rowblock);}
                                            else if(i==6){scanf(argv[6],&structuredSparsity;}
```

