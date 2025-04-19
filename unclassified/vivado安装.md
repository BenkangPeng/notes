参考链接：

[Vivado 安装教程 - Digital Lab 2024 (ustc.edu.cn)](https://soc.ustc.edu.cn/Digital/history/2023/lab0/vivado/)

[vivado2019.1安装 - 简书 (jianshu.com)](https://www.jianshu.com/p/c402a83de9c2)

* `vivado`安装包可从`AMD`官网下载，速度更快。

**license**

将下列字符串保存到文本文档中，并重命名为【vivado_lic2037.lic】

```
INCREMENT VIVADO_HLS xilinxd 2037.05 permanent uncounted AF3E86892AA2 \
	VENDOR_STRING=License_Type:Bought HOSTID=ANY ISSUER="Xilinx \
	Inc" START=19-May-2016 TS_OK
INCREMENT Vivado_System_Edition xilinxd 2037.05 permanent uncounted \
	A1074C37F742 VENDOR_STRING=License_Type:Bought HOSTID=ANY \
	ISSUER="Xilinx Inc" START=19-May-2016 TS_OK
PACKAGE Vivado_System_Edition xilinxd 2037.05 DFF4A65E0A68 \
	COMPONENTS="ISIM ChipScopePro_SIOTK PlanAhead ChipscopePro XPS \
	ISE HLS_Synthesis AccelDSP Vivado Rodin_Synthesis \
	Rodin_Implementation Rodin_SystemBuilder \
	PartialReconfiguration AUTOESL_FLOW AUTOESL_CC AUTOESL_OPT \
	AUTOESL_SC AUTOESL_XILINX petalinux_arch_ppc \
	petalinux_arch_microblaze petalinux_arch_zynq ap_sdsoc SDK \
	SysGen Simulation Implementation Analyzer HLS Synthesis \
	VIVADO_HLS" OPTIONS=SUITE


# 2037年之前的任何Vivado版本（包括HLS、AccelDSP、System Generator、软硬CPU、SOC、嵌入式Linux、重配置等等功能）都是永久使用。使用本license文件时要改名，文件名不能有汉字和空格。
```

```cpp
#include<iostream>
#include<ctime>
#include<cstdlib>
#include<random>
#include<cstdint>
#include "sa_8_8.cpp"
using namespace std;


class matrix{
    private:
        void init(){
            for(int i = 0 ; i < 8 ; i++){
                for(int j = 0 ; j < 8 ; j++){
                    data[i][j] = rand() % 100 - 50; // -50 ~ 50
                }
            }
        }



    public:
        int32_t data[8][8];
        matrix(){
            init();
        }

        void print_matrix(){
            for(int i = 0 ; i < 8 ; ++i){
                for(int j = 0 ; j < 8 ; ++j){
                    std::cout << data[i][j] << ' ';
                }
            
            std::cout << std::endl;
            }
            std::cout << "================" << std::endl;
        }

        
         
};

void gold_gemm(int32_t v0[8][8] , int32_t v1[8][8] , int32_t v2[8][8]){

    for(int i = 0 ; i < 8 ; i++){
        for(int j = 0 ; j < 8 ; j++){
            v2[i][j] = 0;
        }
    }


    for(int i = 0 ; i < 8 ; i++){
        for(int j = 0 ; j < 8 ; j++){
            for(int k = 0 ; k < 8 ; k++){
                v2[i][j] += v0[i][k] * v1[k][j];
            }
        }
    }

    
}

void compare(int32_t a[8][8] , int32_t b[8][8]){
    for(int i = 0 ; i < 8 ; i++){
        for(int j = 0 ; j < 8 ; j++){
            if(a[i][j] != b[i][j]){
                std::cout << "========fail=========";
                return;
            }
        }
    }

    std::cout << "========pass=======" << std::endl;
}
int main()
{
    srand(time(0));
    matrix A , B , C;
    gemm(A.data , B.data , C.data);

    int32_t gold_c[8][8];
    gold_gemm(A.data , B.data , gold_c);

    compare(C.data , gold_c);

    A.print_matrix();
    B.print_matrix();
    C.print_matrix();

    return 0;
}
```

