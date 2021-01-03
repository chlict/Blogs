# MLIR中的standalone工程
MLIR官方提供了一个toy工程来做MLIR的tutorial，其中介绍了toy语言、AST、定义Dialet以及Operations、Lowering。这个工程仍然比较复杂。实际上，*llvm-project/mlir/examples*目录下还有一个用例，叫standalone，相比toy要更简单，我们也可以通过它来学习MLIR的基本能力。

## build 'standalone'
要build standalone，首先要build llvm-project。按照standalone中README.md的说明，在构建llvm-project时要加`-DLLVM_INSTALL_UTILS=ON`选项，目的是为了生成FileCheck这个工具。 
之后遵照README的说明，单独构建standalone工程：
> ```sh
> mkdir build && cd build
> cmake -G Ninja .. -DMLIR_DIR=$PREFIX/lib/cmake/mlir -DLLVM_EXTERNAL_LIT=$BUILD_DIR/bin/llvm-lit
> cmake --build . --target check-standalone
> ```
最后一步时可以增加--verbose选项把详细过程打印出来。共有13个子命令，前面几个子命令调用mlir-tblgen这个工具来生成MLIR所需的各种代码。比如第一步：
```shell
[1/13] cd /Users/chenlong/software/llvm-project/mlir/examples/standalone/build && /Users/chenlong/software/llvm-project/build/bin/mlir-tblgen -gen-typedef-decls -I /Users/chenlong/software/llvm-project/mlir/examples/standalone/include/Standalone -I/Users/chenlong/software/llvm-project/llvm/include -I/Users/chenlong/software/llvm-project/build/include -I/Users/chenlong/software/llvm-project/mlir/include -I/Users/chenlong/software/llvm-project/build/tools/mlir/include -I/Users/chenlong/software/llvm-project/mlir/examples/standalone/include -I/Users/chenlong/software/llvm-project/mlir/examples/standalone/build/include /Users/chenlong/software/llvm-project/mlir/examples/standalone/include/Standalone/StandaloneOps.td --write-if-changed -o include/Standalone/StandaloneOpsTypes.h.inc -d include/Standalone/StandaloneOpsTypes.h.inc.d
```
通过观察各个子命令的输入和输出，可以了解定义一个Dialect以及其Op的主要过程。

|mlir-tablegen选项|输入|输出|
|----|----|----|
|-gen-typedef-decls|StandaloneOps.td|StandaloneOpsTypes.h.inc|
|-gen-typedef-defs|StandaloneOps.td|StandaloneOpsTypes.cpp.inc|
|-gen-dialect-decls -dialect=standalone|StandaloneOps.td|StandaloneOpsDialect.h.inc
|-gen-op-defs|StandaloneOps.td|StandaloneOps.cpp.inc|
|-gen-op-decls|StandaloneOps.td|StandaloneOps.h.inc|

第6步开始用c++编译：

|c++|输入|输出|
|----|----|----|
|c++ -c|standalone-translate.cpp|standalone-translate.cpp.o|
|c++ -c|StandaloneDialect.cpp|StandaloneDialect.cpp.o|
|c++ -c|StandaloneOps.cpp|StandaloneOps.cpp.o|
|c++ |tandalone-translate.cpp.o *.a |standalone-translate|
|c++ -c|standalone-opt.cpp|standalone-opt.cpp.o|
|c++ |standalone-opt.cpp.o *.a|standalone-opt|

## 测试'standalone'
上述过程的最后一步是测试：
```
/Users/chenlong/software/llvm-project/build/bin/llvm-lit -sv /Users/chenlong/software/llvm-project/mlir/examples/standalone/build/test
```
查看standalone-translate的测试脚本：
```shell
$ cat test/Standalone/Output/standalone-translate.mlir.script 
set -o pipefail;{ : 'RUN: at line 1';   /Users/chenlong/software/llvm-project/mlir/examples/standalone/build/bin/standalone-translate --help | /Users/chenlong/software/llvm-project/build/bin/FileCheck /Users/chenlong/software/llvm-project/mlir/examples/standalone/test/Standalone/standalone-translate.mlir; }
```
只是测试了--help的输出，从中可以看出MLIR内置了许多有趣的转换能力，比如*arm-neon-mlir-to-llvmir*，看起来arm-neon，avx512等也是一种MLIR的dialect。
```shell
 $ /Users/chenlong/software/llvm-project/mlir/examples/standalone/build/bin/standalone-translate --help
OVERVIEW: MLIR translation driver

USAGE: standalone-translate [options] <input file>

OPTIONS:

Color Options:

  --color                                              - Use colors in output (default=autodetect)

General options:

  Translation to perform
      --arm-neon-mlir-to-llvmir                           - arm-neon-mlir-to-llvmir
      --arm-sve-mlir-to-llvmir                            - arm-sve-mlir-to-llvmir
      --avx512-mlir-to-llvmir                             - avx512-mlir-to-llvmir
      --deserialize-spirv                                 - deserialize-spirv
      --import-llvm                                       - import-llvm
      --mlir-to-llvmir                                    - mlir-to-llvmir
      --mlir-to-nvvmir                                    - mlir-to-nvvmir
      --mlir-to-rocdlir                                   - mlir-to-rocdlir
      --serialize-spirv                                   - serialize-spirv
```