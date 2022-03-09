# 在 macos m1 下创建 arm 交叉编译工具

## 1. 安装 `crosstool-NG`

```
brew install crosstool-ng
```

当前(2022-03-09)的 homebrew 安装的 crosstool-NG 版本是 1.24.0_3, 需要打些补丁. 如果是新版本可能就不需要了

### 补丁 1: 替换 `scripts/config.guess` 文件

```
cd /opt/homebrew/Cellar/crosstool-ng/1.24.0_3/share/crosstool-ng/scripts

./config.guess
```

如果返回的不是类似 `aarch64-apple-darwin21.3.0`, 而是 `arm` 开头的, 说明需要替换

```
curl https://raw.githubusercontent.com/crosstool-ng/crosstool-ng/master/scripts/config.guess > config.guess
```

### 补丁 2: 添加 gcc 补丁

```
ls /opt/homebrew/Cellar/crosstool-ng/1.24.0_3/share/crosstool-ng/packages/gcc
```

看一下有没有想使用的 gcc 版本号, 如果没有, 去官网下载 [https://github.com/crosstool-ng/crosstool-ng/tree/master/packages/gcc]

## 2. 本机安装一个 gcc

交叉编译的 gcc 也需要一个 gcc 来编译它

```
brew install gcc
```

注意本机上的 gcc 前缀和后缀, 比如在我机器上为 `aarch64-apple-darwin21-gcc-11`

那么前缀就是 `aarch64-apple-darwin21-`, 后缀是 `-11`

这个要与 `.config` 文件中的 `CT_BUILD_PREFIX`, `CT_BUILD_SUFFIX` 保持一致

## 3. 创建编译环境

`crosstool-ng` 的运行需要一个大小写敏感的文件系统, 而 macos 下默认是不敏感的, 所以要创建一个新的 Volume

使用系统带的磁盘工具, 添加 APFS 卷宗, 随便起个名(比如 Linux), 选择 'APFS (区分大小写)' 即可

```
cd /Volumes/Linux
mkdir crosstool
cd crosstool
mkdir tarballs
mkdir x-tool
```

把 `.config` 文件拷贝到 `/Volumes/Linux/crosstool` 中即可

## 4. 配置 .config

可以使用图形化的修改

```
ct-ng menuconfig
```

也可以直接修改 `.config` 文件, 改需要的参数.

下面列出几个重要的

### CT_GMP_VERSION

默认的 GMP 6.1.2 在 m1 下也会有上面的 config.guess 问题, 所以选一个年代比较近的, 这里用 `6.2.99-20220306144058`

### CT_BINUTILS_VERSION

默认的是 2.32, 但有 bug(忘记了), 这里选 2.38

### 其他

gcc 版本 (CT_GCC_VERSION), linux 内核版本(CT_LINUX_VERSION), glibc 版本(CT_GLIBC_VERSION) 根据需要选即可

**注意 1: gcc 的版本一定保证存在 crosstool-ng 补丁, 否则会有很多问题**

**注意 2: 图形化修改后, 一定要和之前的文件(.config.old)比较一下, 它会修改一些默认值, 比如 `CT_GMP_VERSION`**

## 开始运行

```
ulimit -n 2048
ct-ng build
```

成功后文件会放到 `x-tool` 目录中, 也可以整体把目录拷贝到其他地方

测试一下

```
/Volumes/Linux/crosstool/x-tool/bin/arm-unknown-linux-gnueabihf-gcc -v
```
