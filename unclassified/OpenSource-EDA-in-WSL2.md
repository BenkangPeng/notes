**Set up `iverilog + gtkwave + yosys` opensource EDA tools in WSL2 .**

<!-- more -->

#### Install WSL2 :

**Refer to :** 

* [video](https://www.bilibili.com/video/BV1aA411s7PJ/?spm_id_from=333.788.top_right_bar_window_custom_collection.content.click&vd_source=da5120fea3f8bb8d2fe1984a02a9a745)
  tips : must enable the `hype-v` before installing `wsl2`

* [modify wsl2's storage location](https://answers.microsoft.com/zh-hans/windows/forum/all/wsl2%E6%98%AF%E5%90%A6%E5%8F%AF%E4%BB%A5%E5%AE%89/608a35fd-a8d6-40cc-854d-65bead8ef49d)

​    **After installing wsl2 according to video above** 

1. Check the wsl version in powershell : 

```powershell
wsl -l --all -v
```

2. Export the wsl into `.tar` file ,  taking `ubuntu-22.04` for example .

```powershell
wsl --export  Ubuntu-22.04 e:/wsl-ubuntu22.04.tar
```

3. Unregister ubuntu

```powershell
wsl --unregister Ubuntu-22.04
```

4. Import ubuntu (turn `wsl-ubuntu22.04.tar` into `Ubuntu-22.04`)

```powershell
wsl --import Ubuntu-22.04 e:/wsl-ubuntu22.04 e:/wsl-ubuntu22.04.tar --version 2
```

5. Set username

```powershell
ubuntu2204 config --default-user <your username>
```

6. Delete `.tar` file 

```powershell
del e:/wsl-ubuntu22.04.tar
```

#### 

#### Iverilog & gtkwave

* Offical site : [Icarus Verilog documentation](https://steveicarus.github.io/iverilog/)

* Installation

```shell
sudo apt-get install iverilog 
sudo apt-get install gtkwave
```

##### Get started

1. code the `hello.v` :

```verilog
module hello;
  initial
    begin
      $display("Hello, World");
      $finish ;
    end
endmodule
```

```shell
$ iverilog  -o hello hello.v
$ vvp hello
```
