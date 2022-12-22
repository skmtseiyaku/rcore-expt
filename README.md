# rcore-expt
A step-by-step implementation of rCore OS following the official tutorial.
- [rCore-Tutorial-Book-v3](https://rcore-os.cn/rCore-Tutorial-Book-v3/index.html)
## Environment
- Ubuntu 22.04
- QEMU 7.0.0
  - Self-compiled

## Notes

### Local Environment

- All user programs are installed in `~/.local/bin/`.
### Ch. 1 
#### 2. remove-std
  - [ ] x86_64平台上移除标准库依赖

#### 4. first-instruction-in-kernel2
  - [ ] 使用如下命令可以丢弃内核可执行文件中的元数据得到内核镜像：
    ```
    $ rust-objcopy --strip-all path/to/source -O binary path/to/target
    ```
  - [x] 基于 GDB 验证启动流程：
    
    - `bootloader/rustsbi-qemu.bin` put in project root
    - [rustsbi/rustsbi-qemu](https://github.com/rustsbi/rustsbi-qemu)
    - [sifive/freedom-tools](https://github.com/sifive/freedom-tools)  (official prebuilt)
    - [ ] [rustsbi/rustsbi](https://github.com/rustsbi/rustsbi) How-to: build a custom RustSBI firmware for QEMU 

    ```
    $ qemu-system-riscv64 \
      -machine virt \
      -nographic \
      -bios ./bootloader/rustsbi-qemu.bin \
      -device loader,file=./target/riscv64gc-unknown-none-elf/release/rcore-expt.bin,addr=0x80200000 \
      -s -S

    $ riscv64-unknown-elf-gdb \
      -ex 'file ./target/riscv64gc-unknown-none-elf/release/rcore-expt' \
      -ex 'set arch riscv:rv64' \
      -ex 'target remote localhost:1234'
    ```
  - [ ] 0x101a 处的数据 0x8000 是能够跳转到 0x80000000 进入启动下一阶段的关键；自行探究位于 0x1000 和 0x100c 两条指令的含义
    ```
    0x0000000000001000 in ?? ()
    $ (gdb) x/10i $pc
    => 0x1000:      auipc   t0,0x0
       0x1004:      addi    a2,t0,40
       0x1008:      csrr    a0,mhartid
       0x100c:      ld      a1,32(t0)
       0x1010:      ld      t0,24(t0)
       0x1014:      jr      t0
       0x1018:      unimp
       0x101a:      0x8000
       0x101c:      unimp
       0x101e:      unimp
    ```

- 五次`$ (gdb) si`后才能够跳转到`0x80000000`，因为新版本RustSBI所实现的QEMU固件包含的指令增加了一条

#### 5. support-func-call
  - [ ] RISC-V 汇编指令各部分含义

  	> 在大多数只与通用寄存器打交道的指令中，`rs`表示**源寄存器** (Source Register)，`imm`表示**立即数** (Immediate)，是一个常数，二者构成了指令的输入部分；而`rd`表示**目标寄存器** (Destination Register)，它是指令的输出部分。`rs`和`rd`可以在32个通用寄存器`x0~x31`中选取。但是这三个部分都不是必须的，某些指令只有一种输入类型，另一些指令则没有输出部分。

  - [ ] 