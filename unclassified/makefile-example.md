`makefile`是一个编译运行代码的自动化工具，设置`makefile`后能够通过`make run` , `make clean`等命令自动化运行、清除代码。

之前安装`Mingw64`时([Rust notes.md](Rust notes.md)),使用的是`x86_64-13.2.0-release-posix-seh-msvcrt-rt_v11-rev1.7z` , 其中的`make`工具路径和名字是`D:\software\mingw64\bin\mingw32-make.exe`，但我们在命令行中习惯用`make`，可以复制粘贴一份`mingw32-make-副本.exe`,重命名为`make.exe`,就能用`make`命令了。

以下是一个`C++` `makefile`模板

```makefile
CC=g++
CFLAGS=-O2 -Wall
DEBUGFLAGS=-g
SOURCE=draft.cpp
TARGET=draft.out
DEBUG_TARGET=draft_gdb.out

$(TARGET):$(SOURCE)
	$(CC) $(CFLAGS) $(SOURCE) -o $(TARGET)

$(DEBUG_TARGET):$(SOURCE)
	$(CC) $(DEBUGFLAGS) $(SOURCE) -o $(DEBUG_TARGET)
	
all:$(TARGET)


run:$(TARGET)
	./$(TARGET)

debug:$(DEBUG_TARGET)
	gdb $(DEBUG_TARGET)

clean:
	rm -f $(TARGET) $(DEBUG_TARGET)

.PHONY:all run debug clean
```

* `CC=g++`等命令是将命令重命名为`CC`，类似于宏定义，之后便可以用`$(CC)`代替`g++`命令。
* 第二部分的命令非常重要：这里并不是将`$(TARGET)`重命名了。`$(TARGET):$(SOURCE)`的意思是`$(TARGET)`依赖于`$SOURCE`，当`$SOURCE`更新后，将执行`$(CC) $(CFLAGS) $(SOURCE) -o $(TARGET)`来更新`$(TARGET)` ， 若`$SOURCE`没有变化，`$(TARGET)`也不会变化。如果项目`include`了外部的`.h`文件，需要将`.h`添加进来，让`$TARGET`也依赖于`.h`，例如：`$(TARGET):$(SOURCE) my_class.h`。这种写法能够让执行`make run`时，若`$(SOURCE)`没有变化，就不需要重新进行编译，可以直接运行`./main.out`,这对于大型、编译时间长的项目十分有效。

```makefile
$(TARGET):$(SOURCE)
	$(CC) $(CFLAGS) $(SOURCE) -o $(TARGET)
```

* `all:$(TARGET)`表示伪命令`all`也依赖于`$(TARGET)`。`run:$(TARGET)`表示`run`也依赖于`$(TARGET)`，若`$SOURCE`更新，就会重新编译生成`$TARGET`.
* `.PHONY:all run debug clean`指明了当前`makefile`中所有的伪命令。这是为了避免：若当前路径下有名称为`all` `debug`等文件时，`make all`会与之冲突，而用`.PHONY`指出所有伪命令就可以避免这个问题。
* `makefile`严格遵循空格与缩进规则。
* 可以使用`$<`和`$@`来使`makefile`更简洁，其中`$<`表示依赖文件，`$@`表示目标文件。

```makefile
CC=g++
CFLAGS=-O2 -Wall
DEBUG_FLAGS=-g
SOURCE=draft.cpp
TARGET=draft.out
DEBUG_TARGET=debug_draft.out

$(TARGET):$(SOURCE)
	$(CC) $(CFLAGS) $< -o $@

$(DEBUG_TARGET):$(SOURCE)
	$(CC) $(DEBUG_FLAGS) $< -o $@

all:$(TARGET)

run:$(TARGET)
	./$<

debug:$(DEBUG_TARGET)
	gdb $<

clean:
	rm -f $(TARGET) 
	rm -f $(DEBUG_TARGET)

.PHONY:all run debug clean
```



### 更好的`makefile` 规范：

```makefile
CC = g++
CXXFLAGS = -std=c++11 -fno-elide-constructors -Wall
OPTIMIZE_FLAGS = -O0

#declaration in .hpp and defination in .cpp strictly
#SRC equal to all the .cpp file
SRC = redundant-constructor.cpp StringVector.cpp main.cpp emplace_back.cpp

#Replace the file suffix ".cpp" of the files in SRC with ".o".
OBJ = $(SRC:.cpp=.o)

# executable file
EXE1 = redundant-constructor
EXE2 = emplace_back
MAIN_EXE = main

# put all the executable target in `all`
all: $(EXE1) $(EXE2) $(MAIN_EXE)

# Linking the executable
$(EXE1): redundant-constructor.o StringVector.o
	$(CC) $(CXXFLAGS) $(OPTIMIZE_FLAGS) -o $@ $^

$(EXE2): emplace_back.o
	$(CC) -std=c++17 $(OPTIMIZE_FLAGS) -o $@ $^


$(MAIN_EXE): main.o StringVector.o
	$(CC) $(CXXFLAGS) $(OPTIMIZE_FLAGS) -o $@ $^


# Compile each .cpp files into .o files
%.o: %.cpp
	$(CC) $(CXXFLAGS) $(OPTIMIZE_FLAGS) -c $< -o $@

# special version request
emplace_back.o: emplace_back.cpp
	$(CC) -Wall -std=c++17 -c $< -o $@

# Clean up generated files
clean:
	rm -f *.o *.exe

# Phony targets 
.PHONY: all clean
```























