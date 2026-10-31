# tilelang2rvv

真实 TileLang/TVM PrimFunc → RVV intrinsic C → RV64GCV ELF 的独立研究消费器。

**[总项目：tilelang-riscv](https://github.com/zishu0131-png/tilelang-riscv) · [上游项目：numba2tilelang](https://github.com/zishu0131-png/numba2tilelang)**

## 架构与范围

```text
受支持的真实 tvm.tirx.PrimFunc
  → backend.generate
  → RVV intrinsic C
  → Clang / riscv64-unknown-elf / rv64gcv
  → QEMU virt / 单 hart / RVV 1.0
```

支持受限的连续向量访问、动态 VL 尾部裁剪及选定 tile 运算。不是 TileLang 官方注册后端，也不是通用 GPU 模型模拟器：依赖 `threadIdx` 的寻址、跨 hart 同步和未支持 IR 应拒绝。block 顺序执行，shared/fragment 映射为单 hart 局部数组。

上游转换器的线程级生成物**不能自动视为此后端可接受输入**；两个项目独立存在不代表它们所有能力已串联验证。

## 安装与运行

本源码副本沿用 2026-10-29 已公开的 8 件 RVV 文件，未修改算法；文件身份见 `release-manifest.json`。历史研究环境使用 TileLang 0.1.15、Clang 23.1.3、QEMU 11.1.0，不保证其他版本兼容。

```sh
python -m pip install -r requirements.txt
```

另需支持 RVV 的 Clang 与 `qemu-system-riscv64`。PowerShell：

```powershell
$env:RVV_CLANG = "C:\path\to\clang.exe"
$env:RVV_QEMU = "C:\path\to\qemu-system-riscv64.exe"
python -B verify_demo.py
```

Linux/macOS 命令形式（本次未跨平台实测）：

```sh
export RVV_CLANG=/path/to/clang
export RVV_QEMU=/path/to/qemu-system-riscv64
python -B verify_demo.py
```

未配置时使用 Windows 用户目录下的 Scoop 路径回退，不是跨平台自动探测。不要默认设置 `KMP_DUPLICATE_LIB_OK=TRUE` 绕过重复 OpenMP 运行时冲突；应修复依赖环境。

## 示例、产出与测试

`examples.py` 提供 14 个示例及独立 C 参考；`verify_demo.py` 保存 IR、C、汇编、ELF 和 JSON 到 `build/`，逐个运行 VLEN 128/256/512。

```sh
python -B -m unittest -v test_backend test_runner
```

该测试会调用工具链与 QEMU，不是纯 Python 单测。工具缺失引起的 skip 不计通过。检查数值参考、退出码及负控，不只看汇编含 RVV 指令或输出 PASS。

**本次拆仓未重跑 Clang/QEMU 或硬件验证。** 历史 14×3=42 次配置运行不等于 42 个独立算子，也不算本次发布副本的新结果。已公开副本移除了原开发版本默认 OpenMP 绕过，此改动后的 RVV 数值验证尚待重新执行。

## 文件

- `backend.py`：IR 验证与 RVV C 发射。
- `examples.py`：示例和独立参考。
- `runner.py`、`start.S`、`baremetal.ld`：交叉编译与有超时的裸机 QEMU 运行。
- `test_backend.py`、`test_runner.py`：测试和负控。
- [本次验证记录](docs/validation.md)。

未实现或验证 VCIX；QEMU 单 hart 功能结果不能推出真实芯片、多核或性能结论。构建输出、内部交接和设备连接信息不上传。

## 许可

尚未选择项目额外开源许可证；公开可读不等于 MIT/Apache 授权。TileLang/TVM、LLVM/Clang、QEMU 为独立依赖，使用须遵守其许可证，本仓库不打包或代表这些上游项目。代码及发布文档整理使用了 AI 辅助。
