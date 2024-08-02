# Makefile



# CMake

## CMake概述

CMake 是一个项目构建工具，并且是跨平台的。关于项目构建我们所熟知的还有Makefile（通过 make 命令进行项目的构建），**大多是IDE软件都集成了make，比如：VS 的 nmake、linux 下的 GNU make、Qt 的 qmake等**，如果自己动手写 makefile，会发现，makefile 通常依赖于当前的编译平台，而且编写 makefile 的工作量比较大，解决依赖关系时也容易出错。

而 CMake 恰好能解决上述问题， 其允许开发者指定整个工程的编译流程，在根据编译平台，自动生成本地化的Makefile和工程文件，最后用户只需make编译即可，所以可以把**CMake看成一款自动生成 Makefile的工具**，其编译流程如下图：


![image-20230309130644912](https://subingwen.cn/cmake/CMake-primer/image-20230309130644912.png)



- 蓝色虚线表示使用makefile构建项目的过程

- 红色实线表示使用cmake构建项目的过程

介绍完CMake的作用之后，再来总结一下它的优点：

- 跨平台

- 能够管理大型项目
- 简化编译构建过程和编译过程
- 可扩展：可以为 cmake 编写特定功能的模块，扩充 cmake 功能

## CMake的使用

CMake支持大写、小写、混合大小写的命令。如果在编写CMakeLists.txt文件时使用的工具有对应的命令提示，那么大小写随缘即可，不要太过在意。

### 注释

1. 注释行

   > CMake 使用 # 进行行注释，可以放在任何位置。
   >
   > ```cmake
   > # 这是一个 CMakeLists.txt 文件
   > cmake_minimum_required(VERSION 3.0.0)
   > ```
   >
   > 

2. 注释块

   > CMake 使用 #[[ ]] 形式进行块注释
   >
   > ```cmake
   > #[[ 这是一个 CMakeLists.txt 文件。
   > 这是一个 CMakeLists.txt 文件
   > 这是一个 CMakeLists.txt 文件]]
   > cmake_minimum_required(VERSION 3.0.0)
   > 
   > ```
   >
   > 

### 示例

#### 1.具体代码

- head.h

  > ```
  > #ifndef _HEAD_H
  > #define _HEAD_H
  > // 加法
  > int add(int a, int b);
  > // 减法
  > int subtract(int a, int b);
  > // 乘法
  > int multiply(int a, int b);
  > // 除法
  > double divide(int a, int b);
  > #endif
  > ```

- add.c

  > ```
  > #include <stdio.h>
  > #include "head.h"
  > 
  > int add(int a, int b)
  > {
  >     return a+b;
  > }
  > ```
  >
  > 

- sub.c

  > ```
  > #include <stdio.h>
  > #include "head.h"
  > 
  > // 你好
  > int subtract(int a, int b)
  > {
  >     return a-b;
  > }
  > ```

- mult.c

  > ```
  > #include <stdio.h>
  > #include "head.h"
  > 
  > int multiply(int a, int b)
  > {
  >     return a*b;
  > }
  > ```

- div.c

  > ```
  > #include <stdio.h>
  > #include "head.h"
  > 
  > double divide(int a, int b)
  > {
  >     return (double)a/b;
  > }
  > ```

- main.c

  > ```
  > #include <stdio.h>
  > #include "head.h"
  > 
  > int main()
  > {
  >     int a = 20;
  >     int b = 12;
  >     printf("a = %d, b = %d\n", a, b);
  >     printf("a + b = %d\n", add(a, b));
  >     printf("a - b = %d\n", subtract(a, b));
  >     printf("a * b = %d\n", multiply(a, b));
  >     printf("a / b = %f\n", divide(a, b));
  >     return 0;
  > }
  > 
  > ```

  

#### 2.代码架构

```
$ tree
.
├── add.c
├── div.c
├── head.h
├── main.c
├── mult.c
└── sub.c
```

#### 3.添加CMakeLists.txt文件

在上述源文件所在目录下添加一个新文件 CMakeLists.txt，文件内容如下：

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
add_executable(app add.c div.c main.c mult.c sub.c)
```

**接下来介绍一下这三个命令**

1. **cmake_minimum_required**：指定使用的 cmake 的最低版本，非必选，如果不加可能会有**警告**

2. **project**：定义工程名称，并可指定工程的版本、工程描述、web主页地址、支持的语言（默认情况支持所有语言），如果不需要这些都是可以忽略的，只需要指定出工程名字即可。

   ```cmake
   # PROJECT 指令的语法是：
   project(<PROJECT-NAME> [<language-name>...])
   project(<PROJECT-NAME>
          [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
          [DESCRIPTION <project-description-string>]
          [HOMEPAGE_URL <url-string>]
          [LANGUAGES <language-name>...])
   
   ```

   

3. add_executable：定义工程会生成一个可执行程序 定义工程会生成一个可执行程序

   ```cmake
   add_executable(可执行程序名 源文件名称)
   
   #这里的可执行程序名和project中的项目名没有任何关系
   #源文件名可以是一个也可以是多个，如有多个可用空格或;间隔
   
   #示例：
   
   # 样式1
   add_executable(app add.c div.c main.c mult.c sub.c)
   # 样式2
   add_executable(app add.c;div.c;main.c;mult.c;sub.c)
   
   ```

#### 4.执行CMake命令

 CMakeLists.txt 文件编辑好之后，就可以执行 cmake命令了

在shell中使用下面的命令：

```
cmake CMakeLists.txt文件所在路径
```

**一般来说我们会新建一个build目录，然后cd build目录，再使用上面的命令，这样做的目的是让cmake生成的文件不要显得很杂乱，并且删除这些生成的文件也很便捷**

当执行cmake命令之后，CMakeLists.txt 中的命令就会被执行，所以一定要注意给cmake 命令指定路径的时候一定不能出错。

执行命令之后，看一下源文件所在目录中是否多了一些文件：（下面是shell命令输出的内容）

```
$ tree -L 1
.
├── add.c
├── CMakeCache.txt         # new add file
├── CMakeFiles             # new add dir
├── cmake_install.cmake    # new add file
├── CMakeLists.txt
├── div.c
├── head.h
├── main.c
├── Makefile               # new add file
├── mult.c
└── sub.c

```

我们可以看到在对应的目录下生成了一个makefile文件，此时再执行make命令（要和生成的Makefile处于同一目录下哟），就可以对项目进行构建得到所需的可执行程序了。下面是shell命令输出的内容

```
$ make
Scanning dependencies of target app
[ 16%] Building C object CMakeFiles/app.dir/add.c.o
[ 33%] Building C object CMakeFiles/app.dir/div.c.o
[ 50%] Building C object CMakeFiles/app.dir/main.c.o
[ 66%] Building C object CMakeFiles/app.dir/mult.c.o
[ 83%] Building C object CMakeFiles/app.dir/sub.c.o
[100%] Linking C executable app
[100%] Built target app

# 查看可执行程序是否已经生成
$ tree -L 1
.
├── add.c
├── app					# 生成的可执行程序
├── CMakeCache.txt
├── CMakeFiles
├── cmake_install.cmake
├── CMakeLists.txt
├── div.c
├── head.h
├── main.c
├── Makefile
├── mult.c
└── sub.c

```

### 定制cmake，set命令

#### 1.定义变量

在上面的例子中一共提供了5个源文件，假设这五个源文件需要反复被使用，每次都直接将它们的名字写出来确实是很麻烦，此时我们就需要定义一个变量，将文件名对应的字符串存储起来，在cmake里定义变量需要使用set。**如果要使用变量则需要在前加上${变量名}**

```cmake
# SET 指令的语法是：
# [] 中的参数为可选项, 如不需要可以不写
SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])

```

VAR：变量名
VALUE：变量值

```cmake
# 方式1: 各个源文件之间使用空格间隔
set(SRC_LIST add.c  div.c   main.c  mult.c  sub.c)

# 方式2: 各个源文件之间使用分号 ; 间隔
set(SRC_LIST add.c;div.c;main.c;mult.c;sub.c)
add_executable(app  ${SRC_LIST})

```

#### 2.指定使用的c++标准

在编写C++程序的时候，可能会用到C++11、C++14、C++17、C++20等新特性，那么就需要在编译的时候在编译命令中制定出要使用哪个标准：

可以直接在shell中使用如下的命令

```
$ g++ *.cpp -std=c++11 -o app
```

上面的例子中通过参数-std=c++11指定出要使用c++11标准编译程序，**C++标准在CMake中对应有一宏叫做<font color='red'>CMAKE_CXX_STANDARD</font>**。在CMake中想要指定C++标准有两种方式：

1. **<font color='red'>在 CMakeLists.txt 中通过 set 命令指定</font>**

   ```cmake
   #增加-std=c++11
   set(CMAKE_CXX_STANDARD 11)
   #增加-std=c++14
   set(CMAKE_CXX_STANDARD 14)
   #增加-std=c++17
   set(CMAKE_CXX_STANDARD 17)
   
   ```

2. <font color='red'>**在执行 cmake 命令的时候指定出这个宏的值**</font>

   ```cmake
   #增加-std=c++11
   cmake CMakeLists.txt文件路径 -DCMAKE_CXX_STANDARD=11
   #增加-std=c++14
   cmake CMakeLists.txt文件路径 -DCMAKE_CXX_STANDARD=14
   #增加-std=c++17
   cmake CMakeLists.txt文件路径 -DCMAKE_CXX_STANDARD=17
   ```

   注意：<font color='red'>需要根据cmake的例子酌情修改哟</font>

#### 3.指定输出的路径

在CMake中指定可执行程序输出的路径，也对应一个宏，叫做**<font color='red'>EXECUTABLE_OUTPUT_PATH</font>**，它的值还是通过set命令进行设置:

```cmake
set(HOME /home/robin/Linux/Sort)
set(EXECUTABLE_OUTPUT_PATH ${HOME}/bin)
```

第一行：定义一个变量用于存储一个绝对路径
第二行：将拼接好的路径值设置给EXECUTABLE_OUTPUT_PATH宏
**如果这个路径中的子目录不存在，会自动生成，无需自己手动创建**

注：<font color='red'>由于可执行程序是基于 cmake 命令生成的 makefile 文件然后再执行 make 命令得到的，所以如果此处指定可执行程序生成路径的时候使用的是相对路径 ./xxx/xxx，那么这个路径中的 ./ 对应的就是 makefile 文件所在的那个目录</font>。

### 变量操作

#### 1.拼接

**有时候项目中的源文件并不一定都在同一个目录中，但是这些源文件最终却需要一起进行编译来生成最终的可执行文件或者库文件**。如果我们通过file命令对各个目录下的源文件进行搜索，最后还需要做一个字符串拼接的操作，关于字符串拼接可以使用set命令也可以使用list命令。

- 使用set拼接

  > 如果使用set进行字符串拼接，对应的命令格式如下：
  >
  > ```cmake
  > set(变量名1 ${变量名1} ${变量名2} ...)
  > ```
  >
  > 关于上面的命令其实就是将从第二个参数开始往后所有的字符串进行拼接，最后将结果存储到第一个参数中，**如果第一个参数中原来有数据会对原数据就行覆盖**。
  >
  > ```cmake
  > cmake_minimum_required(VERSION 3.0)
  > project(TEST)
  > set(TEMP "hello,world")
  > file(GLOB SRC_1 ${PROJECT_SOURCE_DIR}/src1/*.cpp)
  > file(GLOB SRC_2 ${PROJECT_SOURCE_DIR}/src2/*.cpp)
  > # 追加(拼接)
  > set(SRC_1 ${SRC_1} ${SRC_2} ${TEMP})
  > message(STATUS "message: ${SRC_1}")
  > ```
  >
  > 

- 使用list拼接

  > ```cmake
  > list(APPEND <list> [<element> ...])
  > ```
  >
  > list命令的功能比set要强大，字符串拼接只是它的其中一个功能，所以需要在它**第一个参数的位置指定出我们要做的操作，APPEND**表示进行数据追加，后边的参数和set就一样了。
  >
  > ```cmake
  > cmake_minimum_required(VERSION 3.0)
  > project(TEST)
  > set(TEMP "hello,world")
  > file(GLOB SRC_1 ${PROJECT_SOURCE_DIR}/src1/*.cpp)
  > file(GLOB SRC_2 ${PROJECT_SOURCE_DIR}/src2/*.cpp)
  > # 追加(拼接)
  > list(APPEND SRC_1 ${SRC_1} ${SRC_2} ${TEMP})
  > message(STATUS "message: ${SRC_1}")
  > 
  > ```
  >
  > 在CMake中，使用set命令可以创建一个list。一个在list内部是一个由分号;分割的一组字符串。例如，set(var a b c d e)命令将会创建一个（list:） a;b;c;d;e，但是最终打印变量值的时候得到的是abcde。
  >
  > ```
  > set(tmp1 a;b;c;d;e)
  > set(tmp2 a b c d e)
  > message(${tmp1})
  > message(${tmp2})
  > ```
  >
  > 输出的结果
  >
  > ```
  > abcde
  > abcde
  > ```

#### 2.移除字符串

我们在通过file搜索某个目录就得到了该目录下所有的源文件，但是其中有些源文件并不是我们所需要的，比如：

```
$ tree
.
├── add.cpp
├── div.cpp
├── main.cpp
├── mult.cpp
└── sub.cpp

0 directories, 5 files
```

在当前这么目录有五个源文件，其中main.cpp是一个测试文件。如果我们想要把计算器相关的源文件生成一个动态库给别人使用，那么只需要add.cpp、div.cp、mult.cpp、sub.cpp这四个源文件就可以了。此时，就需要将main.cpp从搜索到的数据中剔除出去，想要实现这个功能，也可以使用list

```
list(REMOVE_ITEM <list> <value> [<value> ...])
```

通过上面的命令原型可以看到删除和追加数据类似，只不过是第一个参数变成了REMOVE_ITEM。

```cmake
cmake_minimum_required(VERSION 3.0)
project(TEST)
set(TEMP "hello,world")
file(GLOB SRC_1 ${PROJECT_SOURCE_DIR}/*.cpp)
# 移除前日志
message(STATUS "message: ${SRC_1}")
# 移除 main.cpp
list(REMOVE_ITEM SRC_1 ${PROJECT_SOURCE_DIR}/main.cpp)
# 移除后日志
message(STATUS "message: ${SRC_1}")

```

可以看到，在第8行把将要移除的文件的名字指定给list就可以了。但是一定要注意通过 **file 命令搜索源文件的时候得到的是文件的绝对路径**（在list中每个文件对应的路径都是一个item，并且都是绝对路径），**那么在移除的时候也要将该文件的绝对路径指定出来才可以**，否是移除操作不会成功。




### 包含源文件

如果一个项目里边的源文件很多，在编写CMakeLists.txt文件的时候不可能将项目目录的各个文件一一罗列出来，这样太麻烦也不现实。所以，在CMake中为我们提供了搜索文件的命令，可以使用<font color='red'>aux_source_directory命令或者file命令</font>。

1. 在 CMake 中使用**aux_source_directory** 命令可以查找某个路径下的所有源文件

   > 命令格式
   >
   > ```cmake
   > aux_source_directory(< dir > < variable >)
   > ```
   >
   > - dir：要搜索的目录
   > - variable：将从dir目录下搜索到的源文件列表存储到该变量中
   >
   > **示例：**
   >
   > ```cmake
   > cmake_minimum_required(VERSION 3.0)
   > project(CALC)
   > include_directories(${PROJECT_SOURCE_DIR}/include)
   > # 搜索 src 目录下的源文件
   > aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/src SRC_LIST)
   > add_executable(app  ${SRC_LIST})
   > ```
   >
   > 

2. 使用**file**命令

   > 命令格式
   >
   > ```cmake
   > file(GLOB/GLOB_RECURSE 变量名 要搜索的文件路径和文件类型)
   > ```
   >
   > **下面是file命令中需要传入的参数，二选一，并且将搜索到的文件名放入到变量名中**
   >
   > - GLOB: 将指定目录下搜索到的满足条件的所有文件名生成一个列表，并将其存储到变量中。
   > - GLOB_RECURSE：递归到变量中。
   >
   > **示例：**
   >
   > 搜索当前目录的src目录下所有的源文件，并存储到变量中
   >
   > ```cmake
   > file(GLOB MAIN_SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
   > file(GLOB MAIN_HEAD ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)
   > 
   > ```
   >
   > <font color='red'>CMAKE_CURRENT_SOURCE_DIR </font>宏表示当前访问的 CMakeLists.txt 文件所在的目录。
   >
   > 关于要搜索的文件路径和类型可加双引号，也可不加:
   >
   > ```cmake
   > file(GLOB MAIN_HEAD "${CMAKE_CURRENT_SOURCE_DIR}/src/*.h")
   > ```
   >
   > 

### 包含头文件

在编译项目源文件的时候，很多时候都需要将源文件对应的头文件路径指定出来，这样才能保证在编译过程中编译器能够找到这些头文件，并顺利通过编译。在CMake中设置要包含的目录也很简单，通过一个命令就可以搞定了，他就是**include_directories**:

```cmake
include_directories(headpath)
```

举例说明，有源文件若干，其目录结构如下：(shell命令)

```
$ tree
.
├── build
├── CMakeLists.txt
├── include
│   └── head.h
└── src
    ├── add.cpp
    ├── div.cpp
    ├── main.cpp
    ├── mult.cpp
    └── sub.cpp

3 directories, 7 files
```

CMakeLists.txt文件内容如下:

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
set(CMAKE_CXX_STANDARD 11)
set(HOME /home/robin/Linux/calc)
set(EXECUTABLE_OUTPUT_PATH ${HOME}/bin/)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
add_executable(app  ${SRC_LIST})
```

其中，第六行指定就是头文件的路径，<font color='red'>PROJECT_SOURCE_DIR</font>宏对应的值就是我们在使用cmake命令时，**后面紧跟的目录，一般是工程的根目录（也就是cmake文件存放的目录）**。

### 制作静态库或者动态库

#### 1.静态库

在cmake中，如果要制作静态库，需要使用的命令如下：

```cmake
add_library(库名称 STATIC 源文件1 [源文件2] ...) 
```

在Linux中，静态库名字分为三部分：lib+库名字+.a，**此处只需要指定出库的名字就可以了，另外两部分在生成该文件的时候会自动填充**。

在Windows中虽然库名和Linux格式不同，但也只需指定出名字即可。

下面有一个目录，需要将src目录中的源文件编译成静态库，然后再使用：(shell)

```
.
├── build
├── CMakeLists.txt
├── include           # 头文件目录
│   └── head.h
├── main.cpp          # 用于测试的源文件
└── src               # 源文件目录
    ├── add.cpp
    ├── div.cpp
    ├── mult.cpp
    └── sub.cpp
```

根据上面的目录结构，可以这样编写CMakeLists.txt文件:

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
add_library(calc STATIC ${SRC_LIST})
```

#### 2.动态库

在cmake中，如果要制作动态库，需要使用的命令如下：

```cmake
add_library(库名称 SHARED 源文件1 [源文件2] ...) 
```

在Linux中，动态库名字分为三部分：lib+库名字+.so，此处只需要指定出库的名字就可以了，另外两部分在生成该文件的时候会自动填充。

在Windows中虽然库名和Linux格式不同，但也只需指定出名字即可。

根据上面静态库的目录结构，可以这样编写CMakeLists.txt文件:

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
add_library(calc SHARED ${SRC_LIST})
```


这样最终就会生成对应的动态库文件**libcalc.so**。

#### 3.指定输出的路径

对于生成的库文件来说和可执行程序一样都可以指定输出路径。

- 适用于动态库：

  > **由于在Linux下生成的动态库默认是有执行权限**的，所以可以按照生成可执行程序的方式去指定它生成的目录：
  >
  > ```cmake
  > cmake_minimum_required(VERSION 3.0)
  > project(CALC)
  > include_directories(${PROJECT_SOURCE_DIR}/include)
  > file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
  > # 设置动态库生成路径
  > set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
  > add_library(calc SHARED ${SRC_LIST})
  > 
  > ```
  >
  > 对于这种方式来说，其实就是通过set命令给**EXECUTABLE_OUTPUT_PATH**宏设置了一个路径，**这个路径就是可执行文件生成的路径**。

- 都适用

  > **由于在Linux下生成的静态库默认不具有可执行权限**，所以在指定静态库生成的路径的时候就不能使用EXECUTABLE_OUTPUT_PATH宏了，而应该使用**LIBRARY_OUTPUT_PATH，这个宏对应静态库文件和动态库文件都适用**。
  >
  > ```cmake
  > cmake_minimum_required(VERSION 3.0)
  > project(CALC)
  > include_directories(${PROJECT_SOURCE_DIR}/include)
  > file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
  > # 设置动态库/静态库生成路径
  > set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
  > # 生成动态库
  > #add_library(calc SHARED ${SRC_LIST})
  > # 生成静态库
  > add_library(calc STATIC ${SRC_LIST})
  > 
  > ```

### 链接库文件

在编写程序的过程中，可能会用到一些系统提供的动态库或者自己制作出的动态库或者静态库文件，cmake中也为我们提供了相关的加载动态库的命令。

**注：link_libraries已经被废弃现在主要使用的是target_link_libraries**

#### 1.CMake链接的命令

##### link_libraries

**通过例子来说明**：

当前目录下文件结构（shell）

```
src
├── add.cpp
├── div.cpp
├── main.cpp
├── mult.cpp
└── sub.cpp
```

现在我们把上面src目录中的add.cpp、div.cpp、mult.cpp、sub.cpp编译成一个**静态库文件libcalc.a**。

测试目录结构如下：

```
$ tree 
.
├── build
├── CMakeLists.txt
├── include
│   └── head.h
├── lib
│   └── libcalc.a     # 制作出的静态库的名字
└── src
    └── main.cpp

4 directories, 4 files

```

**在cmake中，链接静态库的命令如下：**

**link_libraries：**

```cmake
link_libraries(<static lib> [<static lib>...])
```

**参数1**：指定出要链接的静态库的名字
	可以是全名 libxxx.a
	也可以是掐头（lib）去尾（.a）之后的名字 xxx

**参数2-N**：要链接的其它静态库的名字(可以链接多个，**如果静态库A中使用了静态库B的内容，那么再编译可执行程序的时候不能只链接A，还需要把B也链接进去**)

如果该静态库不是系统提供的（自己制作或者使用第三方提供的静态库）可能出现静态库找不到的情况，此时可以将静态库的路径也指定出来：

```cmake
link_directories(<lib path>)
```

这样，修改之后的CMakeLists.txt文件内容如下:

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
# 搜索指定目录下源文件
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
# 包含头文件路径
include_directories(${PROJECT_SOURCE_DIR}/include)
# 包含静态库路径
link_directories(${PROJECT_SOURCE_DIR}/lib)
# 链接静态库
link_libraries(calc)
add_executable(app ${SRC_LIST})
```

##### target_link_libraries：

命令格式如下：

```cmake
target_link_libraries(
    <target> 
    <PRIVATE|PUBLIC|INTERFACE> <item>... 
    [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)

```

**target：指定需要加载动态库的文件的名字**

- 该文件可能是一个源文件
- 该文件可能是一个动态库文件
- 该文件可能是一个可执行文件

**PRIVATE|PUBLIC|INTERFACE：库的访问权限，默认为PUBLIC**

- 如果各个动态库之间没有依赖关系，无需做任何设置，三者没有没有区别，一般无需指定，使用默认的 PUBLIC 即可。

- <font color='red'>动态库的链接具有传递性</font>，如果动态库 A 链接了动态库B、C，动态库D链接了动态库A，此时动态库D相当于也链接了动态库B、C，并可以使用动态库B、C中定义的方法。

  ```cmake
  target_link_libraries(A B C)
  target_link_libraries(D A)
  ```

  **PUBLIC**：在public后面的库会被Link到前面的target中，并且里面的符号也会被导出，提供给第三方使用。
  **PRIVATE**：在private后面的库仅被link到前面的target中，并且终结掉，第三方不能感知你调了啥库
  **INTERFACE**：在interface后面引入的库不会被链接到前面的target中，只会导出符号。

**item：**代表动态库文件的名字

<font color='red'>注：target_link_libraries是放在add_executable之后执行</font>


#### 2.链接动态库

动态库的链接和静态库是完全不同的：

- 静态库会在生成可执行程序的链接阶段被打包到可执行程序中，所以可执行程序启动，静态库就被加载到内存中了。

- 动态库在生成可执行程序的链接阶段不会被打包到可执行程序中，当可执行程序被启动并且调用了动态库中的函数的时候，动态库才会被加载到内存

因此，在cmake中指定要链接的动态库的时候，应该将命令写到生成了可执行文件之后

```cmake
cmake_minimum_required(VERSION 3.0)
project(TEST)
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
# 添加并指定最终生成的可执行程序名
add_executable(app ${SRC_LIST})
# 指定可执行程序要链接的动态库名字
target_link_libraries(app pthread)

```

在target_link_libraries(app pthread)中：

- app: 对应的是最终生成的可执行程序的名字

- pthread：这是可执行程序要加载的动态库，这个库是系统提供的线程库，全名为libpthread.so，在指定的时候一般会掐头（lib）去尾（.so）。

<font color='red'>**注意：如果链接的是第三方动态库，需要进行额外的操作**</font>

> 首先自己生成一个动态库，对应的目录结构
>
> ```
> $ tree 
> .
> ├── build
> ├── CMakeLists.txt
> ├── include
> │   └── head.h            # 动态库对应的头文件
> ├── lib
> │   └── libcalc.so        # 自己制作的动态库文件
> └── main.cpp              # 测试用的源文件
> 
> 3 directories, 4 files
> 
> ```
>
> 假设在测试文件main.cpp中既使用了自己制作的动态库libcalc.so又使用了系统提供的线程库，此时CMakeLists.txt文件可以这样写：
>
> ```cmake
> cmake_minimum_required(VERSION 3.0)
> project(TEST)
> file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
> include_directories(${PROJECT_SOURCE_DIR}/include)
> add_executable(app ${SRC_LIST})
> target_link_libraries(app pthread calc)
> ```
>
> 在第六行中，pthread、calc都是可执行程序app要链接的动态库的名字。当可执行程序app生成之后并执行该文件，会提示有如下错误信息：
>
> ```
> $ ./app 
> ./app: error while loading shared libraries: libcalc.so: cannot open shared object file: No such file or directory
> ```
>
> 这是因为可执行程序启动之后，去加载calc这个动态库，但是不知道这个动态库被放到了什么位置解决动态库无法加载的问题，所以就加载失败了([解决动态库加载失败的问题](https://subingwen.cn/linux/library/#2-4-1-%E5%BA%93%E7%9A%84%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86))，在 CMake 中可以在生成可执行程序之前，**通过命令指定出要链接的动态库的位置，指定静态库位置使用的也是这个命令**：
>
> ```cmake
> link_directories(path)
> ```
>
> 所以修改之后的CMakeLists.txt文件应该是这样的：
>
> ```cmake
> cmake_minimum_required(VERSION 3.0)
> project(TEST)
> file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
> # 指定源文件或者动态库对应的头文件路径
> include_directories(${PROJECT_SOURCE_DIR}/include)
> # 指定要链接的动态库的路径
> link_directories(${PROJECT_SOURCE_DIR}/lib)
> # 添加并生成一个可执行程序
> add_executable(app ${SRC_LIST})
> # 指定要链接的动态库
> target_link_libraries(app pthread calc)
> 
> ```
>
> 通过link_directories指定了动态库的路径之后，在执行生成的可执行程序的时候，就不会出现找不到动态库的问题了。
>
> 
>

<font color='red'>**温馨提示：使用 target_link_libraries 命令就可以链接动态库，也可以链接静态库文件。**</font>

### 日志

在CMake中可以用用户显示一条消息，该命令的名字为message：

```cmake
message([STATUS|WARNING|AUTHOR_WARNING|FATAL_ERROR|SEND_ERROR] "message to display" ...)
```

下面是可选互斥参数（只能选一个或者都不选）：

- (无) ：重要消息

- STATUS ：非重要消息
- WARNING：CMake 警告, 会继续执行
- AUTHOR_WARNING：CMake 警告 (dev), 会继续执行
- SEND_ERROR：CMake 错误, 继续执行，但是会跳过生成的步骤
- FATAL_ERROR：CMake 错误, 终止所有处理过程

CMake的命令行工具会在stdout上显示STATUS消息，在stderr上显示其他所有消息。CMake的GUI会在它的log区域显示所有消息。

CMake警告和错误消息的文本显示使用的是一种简单的标记语言。文本没有缩进，超过长度的行会回卷，段落之间以新行做为分隔符。

```cmake
# 输出一般日志信息
message(STATUS "source path: ${PROJECT_SOURCE_DIR}")
# 输出警告信息
message(WARNING "source path: ${PROJECT_SOURCE_DIR}")
# 输出错误信息
message(FATAL_ERROR "source path: ${PROJECT_SOURCE_DIR}")
```

### 添加宏定义

在进行程序测试的时候，我们可以在代码中添加一些宏定义，通过这些宏来控制这些代码是否生效，如下所示：

```
#include <stdio.h>
#define NUMBER  3

int main()
{
    int a = 10;
#ifdef DEBUG
    printf("我是一个程序猿, 我不会爬树...\n");
#endif
    for(int i=0; i<NUMBER; ++i)
    {
        printf("hello, GCC!!!\n");
    }
    return 0;
}
```

在程序的第七行对DEBUG宏进行了判断，如果该宏被定义了，那么第八行就会进行日志输出，如果没有定义这个宏，第八行就相当于被注释掉了，因此最终无法看到日志输入出（上述代码中并没有定义这个宏）。

为了让测试更灵活，我们可以不在代码中定义这个宏，而是在测试的时候去把它定义出来，**其中一种方式就是在gcc/g++命令中去指定**，如下：

```
$ gcc test.c -DDEBUG -o app
```

**在gcc/g++命令中通过参数 -D指定出要定义的宏的名字，这样就相当于在代码中定义了一个宏，其名字为DEBUG。**

在CMake中我们也可以做类似的事情，对应的命令叫做**add_definitions**:

```cmake
add_definitions(-D宏名称)
```

针对于上面的源文件编写一个CMakeLists.txt，内容如下：

```cmake
cmake_minimum_required(VERSION 3.0)
project(TEST)
# 自定义 DEBUG 宏
add_definitions(-DDEBUG)
add_executable(app ./test.c)
```

### CMake编译release或者Debug

1、通过命令行的方式

     cmake  -DCMAKE_BUILD_TYPE=Debug ..
2、编写cmakeLists.txt文件

```cmake
 set(CMAKE_BUILD_TYPE Debug CACHE STRING "set build type to debug")
 或者

 set(CMAKE_BUILD_TYPE "Debug")
```


### CMake中的预定义宏

下面的列表中为大家整理了一些CMake中常用的宏：

|            宏            | 功能                                                         |
| :----------------------: | ------------------------------------------------------------ |
|    PROJECT_SOURCE_DIR    | 使用cmake命令后紧跟的目录，一般是工程的根目录                |
|    PROJECT_BINARY_DIR    | 执行cmake命令的目录                                          |
| CMAKE_CURRENT_SOURCE_DIR | 当前处理的CMakeLists.txt所在的路径                           |
| CMAKE_CURRENT_BINARY_DIR | target 编译目录                                              |
|  EXECUTABLE_OUTPUT_PATH  | 重新定义目标二进制可执行文件的存放位置                       |
|   LIBRARY_OUTPUT_PATH    | 重新定义目标链接库文件的存放位置                             |
|       PROJECT_NAME       | 返回通过PROJECT指令定义的项目名称                            |
|     CMAKE_BINARY_DIR     | 项目实际构建路径，假设在build目录进行的构建，那么得到的就是这个目录的路径 |

## CMake的流程控制

在 CMake 的 CMakeLists.txt 中也可以进行流程控制，也就是说可以像写 shell 脚本那样进行条件判断和循环。

### 条件判断

关于条件判断其语法格式如下：

```cmake
if(<condition>)
  <commands>
elseif(<condition>) # 可选快, 可以重复
  <commands>
else()              # 可选快
  <commands>
endif()
```

在进行条件判断的时候，如果有多个条件，那么可以写多个elseif，最后一个条件可以使用else，但是开始和结束是必须要成对出现的，分别为：if和endif。

#### 1.基本表达式

```cmake
if(<expression>)
```

如果是基本表达式，expression 有以下三种情况：常量、变量、字符串。

如果是**1, ON, YES, TRUE, Y, 非零值，非空字符串时**，条件判断返回True
如果是 **0, OFF, NO, FALSE, N, IGNORE, NOTFOUND，空字符串**时，条件判断返回False

#### 2.逻辑判断

- NOT

  > ```cmake
  > if(NOT <condition>)
  > ```
  >
  > 其实这就是一个取反操作，如果条件condition为True将返回False，如果条件condition为False将返回

- AND

  > ```cmake
  > if(<cond1> AND <cond2>)
  > ```
  >
  > **如果cond1和cond2同时为True，返回True否则返回False。**

- OR

  > ```cmake
  > if(<cond1> OR <cond2>)
  > ```
  >
  > 如果cond1和cond2两个条件中至少有一个为True，返回True，如果两个条件都为False则返回False。

  True。

#### 3.比较

- 基于数值的比较

  > ```cmake
  > if(<variable|string> LESS <variable|string>)
  > if(<variable|string> GREATER <variable|string>)
  > if(<variable|string> EQUAL <variable|string>)
  > if(<variable|string> LESS_EQUAL <variable|string>)
  > if(<variable|string> GREATER_EQUAL <variable|string>)
  > ```
  >
  > - LESS：如果左侧数值小于右侧，返回True
  > - GREATER：如果左侧数值大于右侧，返回True
  > - EQUAL：如果左侧数值等于右侧，返回True
  > - LESS_EQUAL：如果左侧数值小于等于右侧，返回True
  > - GREATER_EQUAL：如果左侧数值大于等于右侧，返回True

- 基于字符串的比较

  > ```cmake
  > if(<variable|string> STRLESS <variable|string>)
  > if(<variable|string> STRGREATER <variable|string>)
  > if(<variable|string> STREQUAL <variable|string>)
  > if(<variable|string> STRLESS_EQUAL <variable|string>)
  > if(<variable|string> STRGREATER_EQUAL <variable|string>)
  > ```
  >
  > - STRLESS：如果左侧字符串小于右侧，返回True
  > - STRGREATER：如果左侧字符串大于右侧，返回True
  > - STREQUAL：如果左侧字符串等于右侧，返回True
  > - STRLESS_EQUAL：如果左侧字符串小于等于右侧，返回True
  > - STRGREATER_EQUAL：如果左侧字符串大于等于右侧，返回True

#### 4.文件

1. 判断文件或者目录是否存在

   ```cmake
   if(EXISTS path-to-file-or-directory)
   ```

   如果文件或者目录存在返回True，否则返回False。

2. 判断是不是目录

   ```cmake
   if(IS_DIRECTORY path)
   ```

   此处目录的 path 必须是绝对路径
   如果目录存在返回True，目录不存在返回False。

3. 判断是不是软连接

   ```cmake
   if(IS_SYMLINK file-name)
   ```

   此处的 file-name 对应的路径必须是绝对路径
   如果软链接存在返回True，软链接不存在返回False。
   软链接相当于 Windows 里的快捷方式

4. 判断是不是绝对路径

   ```cmake
   if(IS_ABSOLUTE path)
   ```

   关于绝对路径:
       如果是Linux，该路径需要从根目录开始描述
       如果是Windows，该路径需要从盘符开始描述
   如果是绝对路径返回True，如果不是绝对路径返回False。

#### 5.其它

1. 判断某个元素是否在列表中

   > ```cmake
   > if(<variable|string> IN_LIST <variable>)
   > ```
   >
   > - CMake 版本要求：大于等于3.3
   > - 如果这个元素在列表中返回True，否则返回False。

2. 比较两个路径是否相等

   > ```cmake
   > if(<variable|string> PATH_EQUAL <variable|string>)
   > ```
   >
   > - CMake 版本要求：大于等于3.24
   > - 如果这个元素在列表中返回True，否则返回False。
   >
   > 关于路径的比较其实就是另个字符串的比较，如果路径格式书写没有问题也可以通过下面这种方式进行比较：
   >
   > ```cmake
   > if(<variable|string> STREQUAL <variable|string>)
   > ```
   >
   > 我们在书写某个路径的时候，可能由于误操作会多写几个分隔符，比如把/a/b/c写成/a//b///c，此时通过STREQUAL对这两个字符串进行比较肯定是不相等的，但是通过PATH_EQUAL去比较两个路径，得到的结果确实相等的，可以看下面的例子：
   >
   > ```cmake
   > cmake_minimum_required(VERSION 3.26)
   > project(test)
   > 
   > if("/home//robin///Linux" PATH_EQUAL "/home/robin/Linux")
   >     message("路径相等")
   > else()
   >     message("路径不相等")
   > endif()
   > 
   > message(STATUS "@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@")
   > 
   > if("/home//robin///Linux" STREQUAL "/home/robin/Linux")
   >     message("路径相等")
   > else()
   >     message("路径不相等")
   > endif()
   > 
   > ```
   >
   > 输出的日志信息如下:
   >
   > ```cmake
   > 路径相等
   > @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
   > 路径不相等
   > ```
   >
   > 通过得到的结果我们可以得到一个结论：在进行路径比较的时候，如果使用 PATH_EQUAL 可以自动剔除路径中多余的分割线然后再进行路径的对比，使用 STREQUAL 则只能进行字符串比较。

### 循环

#### foreach

使用 foreach 进行循环，语法格式如下：

```cmake
foreach(<loop_var> <items>)
    <commands>
endforeach()
```

通过foreach我们就可以对items中的数据进行遍历，然后通过loop_var将遍历到的当前的值取出，在取值的时候有以下几种用法：

1. 方法1：

   > ```cmake
   > foreach(<loop_var> RANGE <stop>)
   > ```
   >
   > - RANGE：关键字，表示要遍历范围
   > - stop：这是一个正整数，表示范围的结束值，在遍历的时候从 0 开始，最大值为 stop。
   > - loop_var：存储每次循环取出的值
   >
   > 例子：
   >
   > ```cmake
   > cmake_minimum_required(VERSION 3.2)
   > project(test)
   > # 循环
   > foreach(item RANGE 10)
   >     message(STATUS "当前遍历的值为: ${item}" )
   > endforeach()
   > ```
   >
   > 输出的日志信息是这样的：
   >
   > ```cmake
   > $ cmake ..
   > -- 当前遍历的值为: 0
   > -- 当前遍历的值为: 1
   > -- 当前遍历的值为: 2
   > -- 当前遍历的值为: 3
   > -- 当前遍历的值为: 4
   > -- 当前遍历的值为: 5
   > -- 当前遍历的值为: 6
   > -- 当前遍历的值为: 7
   > -- 当前遍历的值为: 8
   > -- 当前遍历的值为: 9
   > -- 当前遍历的值为: 10
   > -- Configuring done
   > -- Generating done
   > -- Build files have been written to: /home/robin/abc/a/build
   > ```

   **再次强调：在对一个整数区间进行遍历的时候，得到的范围是这样的 【0，stop】，右侧是闭区间包含 stop 这个值。**

2. 方法2：

   > ```cmake
   > foreach(<loop_var> RANGE <start> <stop> [<step>])
   > ```
   >
   > 这是上面方法1的加强版，我们在遍历一个整数区间的时候，除了可以指定起始范围，还可以指定步长。
   >
   > - RANGE：关键字，表示要遍历范围
   >
   > - start：这是一个正整数，表示范围的起始值，也就是说最小值为 start
   > - stop：这是一个正整数，表示范围的结束值，也就是说最大值为 stop
   > - step：控制每次遍历的时候以怎样的步长增长，默认为1，可以不设置
   > - loop_var：存储每次循环取出的值
   >
   > 举例说明：
   >
   > ```cmake
   > cmake_minimum_required(VERSION 3.2)
   > project(test)
   > 
   > foreach(item RANGE 10 30 2)
   >     message(STATUS "当前遍历的值为: ${item}" )
   > endforeach()
   > 
   > ```
   >
   > 输出的结果如下:
   >
   > ```cmake
   > $ cmake ..
   > -- 当前遍历的值为: 10
   > -- 当前遍历的值为: 12
   > -- 当前遍历的值为: 14
   > -- 当前遍历的值为: 16
   > -- 当前遍历的值为: 18
   > -- 当前遍历的值为: 20
   > -- 当前遍历的值为: 22
   > -- 当前遍历的值为: 24
   > -- 当前遍历的值为: 26
   > -- 当前遍历的值为: 28
   > -- 当前遍历的值为: 30
   > -- Configuring done
   > -- Generating done
   > -- Build files have been written to: /home/robin/abc/a/build
   > ```
   >
   > 再次强调：在使用上面的方式对一个整数区间进行遍历的时候，得到的范围是这样的 【start，stop】，左右两侧都是闭区间，包含 start 和 stop 这两个值，步长 step 默认为1，可以不设置。
   >

3. 方法3：

   > ```cmake
   > foreach(<loop_var> IN [LISTS [<lists>]] [ITEMS [<items>]])
   > ```
   >
   > 这是foreach的另一个变体，通过这种方式我们可以对更加复杂的数据进行遍历，前两种方式只适用于对某个正整数范围内的遍历。
   >
   > - IN：关键字，表示在 xxx 里边
   >
   > - LISTS：关键字，对应的是列表list，通过set、list可以获得
   >
   > - ITEMS：关键字，对应的也是列表
   >
   > - loop_var：存储每次循环取出的值
   >
   > 示例：
   >
   > ```cmake
   > cmake_minimum_required(VERSION 3.2)
   > project(test)
   > # 创建 list
   > set(WORD a b c d)
   > set(NAME ace sabo luffy)
   > # 遍历 list
   > foreach(item IN LISTS WORD NAME)
   >     message(STATUS "当前遍历的值为: ${item}" )
   > endforeach()
   > ```
   >
   > 在上面的例子中，创建了两个 list 列表，在遍历的时候对它们两个都进行了遍历（可以根据实际需求选择同时遍历多个或者只遍历一个）。输出的日志信息如下：
   >
   > ```cmake
   > SHELL
   > $ cd build/
   > $ cmake ..
   > -- 当前遍历的值为: a
   > -- 当前遍历的值为: b
   > -- 当前遍历的值为: c
   > -- 当前遍历的值为: d
   > -- 当前遍历的值为: ace
   > -- 当前遍历的值为: sabo
   > -- 当前遍历的值为: luffy
   > -- Configuring done
   > -- Generating done
   > -- Build files have been written to: /home/robin/abc/a/build
   > ```
   >
   > **一共输出了7个字符串，说明遍历是没有问题的。接下来看另外一种方式：**
   >
   > ```cmake
   > cmake_minimum_required(VERSION 3.2)
   > project(test)
   > 
   > set(WORD a b c "d e f")
   > set(NAME ace sabo luffy)
   > foreach(item IN ITEMS ${WORD} ${NAME})
   >     message(STATUS "当前遍历的值为: ${item}" )
   > endforeach()
   > ```
   >
   > 

4. 方法4：



## 嵌入式cmake



https://subingwen.cn/cmake/CMake-advanced/

## CMake升级

### 1.centos7.9升级cmake为3.0

使用yum安装的cmake等级大多在2.8左右，现在需要3.0以上的版本



> 下载Cmake
>
> wget https://cmake.org/files/v3.0/cmake-3.0.0.tar.gz
>
> 解压Cmake
>
> tar xvf cmake-3.0.0.tar.gz && cd cmake-3.0.0/
>
> 编译安装cmake
>
> ./bootstrap
> gmake
> gmake install
>
> 查看编译后的cmake版本
>
> /usr/local/bin/cmake --version
>
> 移除原来的cmake版本
>
> yum remove cmake -y
>
> 新建软连接
>
> ln -s /usr/local/bin/cmake /usr/bin/
>
> 终端查看版本
>
> cmake --version

## vs code远程使用cmake进行编译项目

下面是我的项目目录

```shell
.
├── CMakeLists.txt
├── easyServer.hpp
├── netWorkMessage.h
└── server_1.5.cpp

0 directories, 4 files
```

使用的g++命令

```cmake
g++ -std=c++11 -lpthread server_1.5.cpp -o server
```

替换为

## windows下vs使用cmake进行编译