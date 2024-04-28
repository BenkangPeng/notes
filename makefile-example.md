`makefile`是一个编译运行代码的自动化工具，设置`makefile`后能够通过`make run` , `make clean`等命令自动化运行、清除代码。

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
* 第二部分的命令非常重要：这里并不是将`$(TARGET)`重命名了。`$(TARGET):$(SOURCE)`的意思是`$(TARGET)`依赖于`$SOURCE`，当`$SOURCE`更新后，将执行`$(CC) $(CFLAGS) $(SOURCE) -o $(TARGET)`来更新`$(TARGET)` ， 若`$SOURCE`没有变化，`$(TARGET)`也不会变化。如果项目`include`了外部的`.h`文件，也可以将`.h`添加进来，让`$TARGET`也依赖于`.h`，例如：`$(TARGET):$(SOURCE) my_class.h`。这种写法能够让执行`make run`时，若`$(SOURCE)`没有变化，就不需要重新进行编译，可以直接运行`./main.out`,这对于大型、编译时间长的项目十分有用。

```makefile
$(TARGET):$(SOURCE)
	$(CC) $(CFLAGS) $(SOURCE) -o $(TARGET)
```

* `all:$(TARGET)`表示伪命令`all`也依赖于`$(TARGET)`。`run:$(TARGET)`表示`run`也依赖于`$(TARGET)`，若`$SOURCE`更新，就会重新编译生成`$TARGET`.
* `.PHONY:all run debug clean`指明了当前`makefile`中所有的伪命令。这是为了避免：若当前路径下有名称为`all` `debug`等文件时，`make all`会与之冲突，而用`.PHONY`指出所有伪命令就可以避免这个问题。

* `makefile`严格遵循空格与缩进规则。