[GitHub - TortoiseFuzz/TortoiseFuzz](https://github.com/TortoiseFuzz/TortoiseFuzz)

# 环境

* Ubuntu 16.04 64位
* LLVM 6.0
* 测试应用程序代码：https://drive.google.com/open?id=1y9DqUCIt0TwS1hJ9TPp7OGtrnK60aP4J

## LLVM

下载llvm和clang 6.0.0源码：

```bash
$ wget https://releases.llvm.org/6.0.0/llvm-6.0.0.src.tar.xz
$ wget https://releases.llvm.org/6.0.0/cfe-6.0.0.src.tar.xz
```

解压下载的压缩包：

```bash
$ tar -xvf llvm-6.0.0.src.tar.xz && mv llvm-6.0.0.src llvm
$ tar -xvf cfe-6.0.0.src.tar.xz && mv cfe-6.0.0.src llvm/tools/clang
```

编译clang。必须有`-DLLVM_ENABLE_ASSERTIONS=On`此选项，否则TortoiseFuzz可能不能正常工作。

```bash
$ mkdir build & cd build
$ cmake -G "Unix Makefiles" -DLLVM_ENABLE_ASSERTIONS=On -DCMAKE_BUILD_TYPE=Release ../
$ make -j4
$ make install
```

将clang 6.0.0添加到PATH环境变量中

```bash
$ export PATH=path_to_clang/build/bin/:$PATH
```

# 安装TortoiseFuzz

```bash
$ git clone https://github.com/TortoiseFuzz/TortoiseFuzz.git
$ cd TortoiseFuzz
$ make
```

# 使用

## 手动

:happy: 这里以bb_metric为例：

1. 首先编译目标应用程序：

   ```bash
    CC=/path_to_TF/bb_metric/afl-clang-fast \
    CXX=/path_to_TF/bb_metric/afl-clang-fast++ \
    ./configure \
    --prefix=/path_to_compiled_program
   ```

2. 启动fuzz，使用 -s 参数去激活工具

   ```bash
   /path_to_TF/bb_metric/afl-fuzz -s -i in -o out_bb -- /path_to_compiled_program ...
   ```

## 自动

1. 编译目标程序：`compile.py`脚本能够自动编译目标程序，编译过程中的参数和TortoiseFuzz的路径应在*compile_arg.json*文件中定义。下面是一个简单的json文件，为了编译`libtiff`程序，使用者应该提供三个参数

   * 第一个参数决定了如何make build文件夹，在大多数情况下，mkdir是足够的，但是你可能需要使用`cp`来复制文件夹
   * 第二个参数决定了编译环境，例如首先使用`cmake`或`conf`然后再make，或者直接`make`
   * 第三个参数是额外的标志，像`-m32`编译32位程序，`空`则默认编译64位程序

   ```json
   [{
   "libtiff"     : ["mkdir", "conf", "-m32"]
   },
   {
   "tofuzz_path" : [absolute_path_to_tofuzz]
   }]
   ```

   为了使用这个脚本，文件应该位于

   ```
   ├── libtiff
   │   └── code
   ├── compile.py
   ```

   使用下面命令行运行该脚本

   ```bash
   $ python compile.py evaluation/compile_arg.json PROGRAM_NAME
   ```

2. 开始模糊测试：fuzz_tf.py能够自动在tmux启动模糊测试进程，目标程序的参数和AFL设置在*fuzz_arg.json*文件中，json格式如下：

   ```json
   [
       {
           "PACKAGE": [ADDITION_AFL_ARG, PROGRAM_ARG], 
       },
       {
           "tofuzz_path" : [absolute_path_to_tofuzz]
       }
   ]
   ```

   `ADDITION_AFL_ARG`是fuzzer的额外参数，如`-m 1000`；`PROGRAM_ARG`是被测程序的命令，对于catdoc来说是`catdoc @@`.为了使用这个脚本，文件位置应如下布局：

   ```
       ├── catdoc
       │   ├── bin_bb
       │   │   └── bin
       │   │       └── catdoc
       │   ├── bin_func     
       │   ├── bin_loop     
       │   └── in                      // the init seeds should be here
       └── fuzz_tf.py
   ```

   使用该脚本的命令行如下：

   ```bash
   $ python fuzz_tf.py evaluation/fuzz_arg.json PROGRAM_NAME
   ```

