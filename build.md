## 安装与使用

### 安装调试器插件

进入插件商店，搜索并下载调试器插件。（需要上传一个现在的版本）

### 开发调试器插件

##### 一、安装开发环境

1.  **安装 Node.js 和 npm**

    在 Ubuntu 上执行以下命令安装 Node.js 和 npm（尽量安装较新的版本）：

    ```shell
    curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    sudo apt-get install -y nodejs
    # 验证版本
    node --version
    npm --version
    ```

2.  **安装rust开发环境**

    ```shell
    sudo apt install curl //要用apt安装curl
    curl https://sh.rustup.rs -sSf | sh
    source $HOME/.cargo/env
    rustup install nightly
    rustup default nightly
    ```

3.  **安装QEMU**

    ```shell
    # 安装编译所需的依赖包
    sudo apt install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev \
                  gawk build-essential bison flex texinfo gperf libtool patchutils bc \
                  zlib1g-dev libexpat-dev pkg-config  libglib2.0-dev libpixman-1-dev libsdl2-dev \
                  git tmux python3 python3-pip ninja-build coreutils xautomation xdotool
    ```

    下载并编译QEMU：

    ```shell
    wget https://download.qemu.org/qemu-7.0.0.tar.xz
    ```

    注：如果下载速度过慢可以使用我们提供的百度网盘链接：<https://pan.baidu.com/s/1dykndFzY73nqkPL2QXs32Q>

    提取码：jimc

    ```shell
    # 解压
    tar xvJf qemu-7.0.0.tar.xz

    # 编译安装并配置 RISC-V 支持
    cd qemu-7.0.0
    ./configure --target-list=riscv64-softmmu,riscv64-linux-user  # 如果要支持图形界面，可添加 " --enable-sdl" 参数
    make -j$(nproc)
    ```

    配置 QEMU 环境变量：

    ```shell
    #编辑~/.bashrc文件，在最后一行添加下面语句：
    export PATH=$PATH:/path/to/qemu-7.0.0/build
    # 注意，执行以上操作时，不能直接复制粘贴,要把/path/to改成qemu所在的文件夹。
    # 另外，执行完以上操作后，要重启终端才能成功添加环境变量。若配置qemu失败，不妨输入$PATH查看环境变量有没有正确添加。
    ```

    ```shell
    # 此时我们可以确认 QEMU 的版本：
    qemu-system-riscv64 --version
    qemu-riscv64 --version
    ```

4.  **安装 RISC-V 工具链**

    在[sifive官网](https://www.sifive.com/software)下载risc-v工具链或者试试直接访问[这里](https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2020.04.1-x86_64-linux-ubuntu14.tar.gz)。下载后将该文件复制到`/home/你的用户名`目录下并解压，将其中的bin/文件夹加入环境变量。

    ```shell
    # 注：Sifive官网提供的工具链没有python支持，因此，如果想用ebpf side-stub，我们要自己编译一份工具链：(更多信息见[riscv-collab/riscv-gnu-toolchain#925](https://github.com/riscv-collab/riscv-gnu-toolchain/issues/925))
    sudo apt install python-is-python3

    sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev ninja-build

    sudo apt install python3-dev

    git clone https://github.com/riscv/riscv-gnu-toolchain

    cd riscv-gnu-toolchain

    ./configure --prefix=/opt/riscv

    sudo make

    # 编译完成后，
    /opt/riscv/bin/riscv64-unknown-elf-gdb

    (gdb) python

    print("114514")

    end

    # 如果gdb输出114514就表示有python支持。
    # 最后将/opt/riscv/bin加入PATH。如果之前将sifive提供的工具链也加入了PATH，应该要把它去掉。
    # 如果你的python没有pyserial模块，应该安装一下：
    pip3 install pyserial

    # 如果想用venv, 可以参考这篇文章。
    ```

5.  **克隆相关仓库**

    克隆 code-debug 和 rcore-ebpf 仓库：

    ```shell
    git clone https://github.com/chenzhiy2001/code-debug	# 建议下载到/home/你的用户名
    git clone --recursive https://github.com/chenzhiy2001/rcore-ebpf	# 建议下载到/home/你的用户名，下载之后跑一遍rcore-ebpf，如下。
    ```

6.  编译rcore-ebpf：

    *   修改user/ebpf/build.sh中里builddir\*\*、\*\*targetdir **和** ucoredir 的路径，将`oslab`改为你实际的用户名；

    *   在 `os` 目录下 `make run`；

    *   编译常见问题：

        *   如果遇到需要更高的nightly版本但是更新后仍出现此错误 在rust-toolchain.toml中将channel改为更新后的版本；

        *   如果在编译过程中遇到“找不到clang-12”报错，执行下面命令：

            ```shell
            sudo apt-get install clang-12
            ```

        *   安装cmake 命令：

            ```shell
            sudo apt-get install cmake
            ```

        *   如果遇到“ riscv64-linux-musl-gcc未找到” [在这里下载](https://git.musl-libc.org/cgit/musl/snapshot/musl-1.2.5.tar.gz) riscv64-linux-musl-cross.tgz 并解压到主目录，将riscv64-linux-musl-cross/bin 加入环境变量：

            ```shell
            nano ~/.bashrc	#打开shell配置文件，在该文件最后添加下面的环境变量：
            export PATH=$PATH:/home/path/to/riscv64-linux-musl-cross/bin
            #添加完成后保存并退出，然后运行以下命令使更改生效：
            source ~/.bashrc
            ```

7.  在 code-debug 文件夹内，执行以下命令**安装依赖**：

    ```shell
    cd code-debug
    npm install
    ```

8.  在vscode中打开项目，按F5执行，会弹出一个新的窗口，即加载了调试器插件。

### 使用调试器插件

1.  在vscode窗口中打开rcore-ebpf项目，在 .vscode 文件夹中添加 launch.json文件，并输入以下内容，然后保存并再编译一遍rCore，接着在新窗口内按F5就可以启动gdb并调试。

    如果GDB并没有正常启动，可以尝试把下面的gdbpath改成绝对路径(例如“/home/username/riscv64-unknown-elf-toolchain-10.2.0-2020.12.8-x86\_64-linux-ubuntu14/bin”)。

    ```shell
    {
      "configurations": [
        {
          "name": "Launch eBPF Debugger",
          "type": "gdb",
          "request": "launch",
          "target": "${workspaceFolder}/path_to_executable",
          "gdbpath": "/path/to/riscv64-unknown-elf-gdb",	#这里的具体路径根据自身情况进行修改
          "setupCommands": [
            {
              "text": "target remote localhost:1234"
            },
            {
              "text": "monitor reset halt"
            }
          ]
        }
      ]
    }
    ```

*   这里解释一下`KERNEL_IN_BREAKPOINTS_LINE`和`GO_TO_KERNEL_LINE`的区别。以rCore-Tutorial-v3为例，`KERNEL_IN_BREAKPOINTS_LINE`对应`trap_return`函数的断点，而`GO_TO_KERNEL_LINE`对应`trap_return`函数调用的`set_user_trap_entry`子函数的断点。而`set_user_trap_entry`子函数实际上只有一行语句：`stvec::write(TRAMPOLINE as usize, TrapMode::Direct);`。也就是说，`KERNEL_IN_BREAKPOINTS_LINE`指向中断处理例程，而`GO_TO_KERNEL_LINE`精确地指向中断处理例程中的`stvec::write(TRAMPOLINE as usize, TrapMode::Direct);`语句。

1.  为了用eBPF Panel，需要在rCore-Tutorial-v3的根目录下添加一个脚本：

```shell
tty > ./qemu_tty
qemu-system-riscv64 "$@" | tee ./code_debug_qemu_output_history.txt
```

将这个脚本命名为`qemu-system-riscv64-with-logs.sh`，添加可执行权限（`chmod +x qemu-system-riscv64-with-logs.sh`），然后将刚才launch.json中的`"qemuPath": "qemu-system-riscv64"`改为`"qemuPath": "${workspaceRoot}/qemu-system-riscv64-with-logs.sh"`.

1.  （可选）如果你要用rust-gdb，先保证你的GDB有Python支持（前文有介绍怎么添加Python支持）然后在rCore-Tutorial-v3的根目录下添加一个脚本：

```shell
export RUST_GDB=riscv64-unknown-elf-gdb
rust-gdb "$@"
```

将这个脚本命名为`riscv64-unknown-elf-gdb-rust.sh`，添加可执行权限，然后将刚才launch.json中的`"gdbpath": "riscv64-unknown-elf-gdb"`改为 `"gdbpath": "${workspaceRoot}/riscv64-unknown-elf-gdb-rust.sh"`.

`RUST_GDB=riscv64-unknown-elf-gdb`（一种方法是，在\~/.bashrc里添加一行`export RUST_GDB=riscv64-unknown-elf-gdb`）然后将launch.json里的gdbpath改为`rust-gdb`.
