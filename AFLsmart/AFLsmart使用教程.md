# AFLsmart

## 安装

安装automake和一些必须的软件包

```bash
sudo apt-get install build-essential automake libtool libc6-dev-i386 python-pip g++-multilib
```

注：其中python-pip不支持ubuntu20（:question: 可以忽略吗？）

建议使用ubuntu16或18来运行AFLsmart :exclamation::exclamation::exclamation:

编译并安装mono包以在Linux上支持C#

```bash
sudo apt-get install mono-complete
```

安装gcc-4.4和g++-4.4（因为Peach中Pin组件在新版本gcc（如gcc-5.4）中存在编译问题）

```bash
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update
sudo apt install gcc-4.4
sudo apt install g++-4.4
```

（上述方法不可用）

如果您发现它在您的linux发行版上不起作用，并且在安装gcc / g ++ 4.4时失败，请按照以下命令操作

首先编辑 sources.list 文件：

```bash
sudo vim /etc/apt/sources.list
```

然后将下面内容添加到您的 sources.list 文件中：

```bash
deb http://dk.archive.ubuntu.com/ubuntu/ trusty main universe
```

然后，

```bash
sudo apt-get update
sudo apt install gcc-4.4
sudo apt install g++-4.4
```

https://peachtech.gitlab.io/peach-fuzzer-community/)

## AFLsmart

建议这里重新编译下面这行

```bash
sudo apt-get install build-essential automake libtool libc6-dev-i386 python-pip g++-multilib
```

下载AFLsmart并编译它

```bash
git clone https://github.com/aflsmart/aflsmart
cd aflsmart
make clean all
cd ..

export AFLSMART=$(pwd)/aflsmart
export WORKDIR=$(pwd)
```

## 修改Peach的版本

```bash
cd $AFLSMART
wget https://sourceforge.net/projects/peachfuzz/files/Peach/3.0/peach-3.0.202-source.zip
unzip peach-3.0.202-source.zip
patch -p1 < peach-3.0.202.patch
cd peach-3.0.202-source
CC=gcc-4.4 CXX=g++-4.4 ./waf configure
CC=gcc-4.4 CXX=g++-4.4 ./waf install
```

## 设置环境变量

```bash
cd $AFLSMART
source $AFLSMART/setup_env.sh
```

## 用法

AFLsmart为AFL增加了四个选项

`-w`：输入模型类型，AFLsmart目前仅支持Peach

`-g`：输入模型文件。输入模型文件（Peach Pits）的路径是必需的。我们在input_models文件夹中提供了10个Peach Pits样例。要为新的文件格式编写新的Peach Pit，请按照本教程并重新访问AFLsmart论文的第4节。

`-h`：将正常（位级别变异）和高阶（块级别变异）变异算子混合在一起的叠加变异模式、

`-H`：限制每个输入的高阶变异树，这是一个可选选项；如果未设计该选项，则表示没有限制。

示例：

```shell
afl-fuzz -h -i in -o out -w peach -g <input model file> -x <dictionary file> <executable binary and its arguments> @@
```

在模糊测试过程中，AFLsmart将与Peach进行交互，以获取有效性和块的边界信息。请检查 out/chunks 文件夹，并确保它不是空的。如果它是空的，则可能无法找到Peach可执行文件，您需要编译Peach并且/或检查PATH环境变量。

## 例子

To fuzz WavPack and reproduce CVE-2018-10536. See [Section 2 - Motivating Example] in the AFLSmart paper.

## 编译有漏洞版本的WavPack

```bash
cd $WORKDIR
git clone https://github.com/dbry/WavPack.git
cd WavPack
git checkout 0a72951
./autogen.sh
CC=afl-gcc ./configure --disable-shared
make clean all
```

## 对WavPack进行24小时模糊测试

```bash
cd $WORKDIR/WavPack
timeout 24h $AFLSMART/afl-fuzz -m none -h -d -i $AFLSMART/testcases/aflsmart/wav -o out -w peach -g $AFLSMART/input_models/wav.xml -x $AFLSMART/dictionaries/wav.dict -e wav -- ./cli/wavpack -y @@ -o out
```

