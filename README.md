# rcore-expt
A step-by-step implementation of rCore OS following the official tutorial.

## Environment
- Ubuntu 22.04
- QEMU 7.0.0
  - Installed in ~/.local/bin

## Notes

### Local Environment

- User programs installed in ~/.local/bin/
### Ch. 1 
#### 2. remove-std
  - [ ] x86_64平台上移除标准库依赖

#### 4. first-instruction-in-kernel2
  - [ ] 使用如下命令可以丢弃内核可执行文件中的元数据得到内核镜像：
    ```
    $ rust-objcopy --strip-all path/to/source -O binary path/to/target
    ```
  - [x] 基于 GDB 验证启动流程：
    ```
    # bootloader/rustsbi-qemu.bin put in project root
    # git@github.com:rustsbi/rustsbi-qemu.git
    # git@github.com:sifive/freedom-tools.git  (official prebuilt)

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
  - [ ] 0x101a 处的数据 0x8000 是能够跳转到 0x80000000 进入启动下一阶段的关键；自行探究位于 0x1000 和 0x100c 两条指令的含义。
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

- 五次```$ (gdb) si```后才能够跳转到```0x80000000```，因为新版本RustSBI实现的QEMU固件包含的指令增加了一条。
