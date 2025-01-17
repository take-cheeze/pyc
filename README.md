> Note: Please do not send pull requests to this repository.


# pyc - Python to LLVM IR transpiler & compiler

```
              🚀 Benchmark Results              
┏━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━┓
┃ runtime        ┃ time              ┃ result  ┃
┡━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━┩
│ LLVM           │ 24.53ms (x1.00)   │ 9227465 │
│ C              │ 36.51ms (x1.49)   │ 9227465 │
│ Python(pyc)    │ 44.34ms (x1.81)   │ 9227465 │
│ Bun            │ 57.44ms (x2.34)   │ 9227465 │
│ Deno           │ 72.81ms (x2.97)   │ 9227465 │
│ Node.js        │ 98.80ms (x4.03)   │ 9227465 │
│ Python         │ 631.09ms (x25.73) │ 9227465 │
│ Python(no GIL) │ 893.46ms (x36.42) │ 9227465 │
└────────────────┴───────────────────┴─────────┘
```

## プロジェクト概要

- **pyc**

  PythonコードをLLVM IRに変換し、最終的にマシンコードへコンパイルするためのメイン機能を提供するモジュールです。以下のサブディレクトリ・ファイルに分かれています:

  - `pyc/__init__.py`

    プロジェクトのバージョン情報やトップレベル関数(`get_codegen`, `get_compiler`)を定義しています。

  - `pyc/__main__.py`

    CLIエントリポイント。コマンドライン引数（`--emit-llvm`, `--compile`, `--dump-ast`など）を解析し、対応する処理を呼び出します。

  - `pyc/codegen/`

    LLVM IR 生成を担うサブモジュール。`generator.py`や`dump_ast.py`などを通じて、Python AST から LLVM IR を生成します。

    - `ir/`

      IRを構築するための`IRBuilder`を定義し、文字列や定数などをグローバルに保持しながらLLVM IRを組み立てる仕組みを提供します。

    - `stdlib/`

      独自に実装したCコードなどが含まれ、`print`, `str`, `int` オブジェクトなどの標準的なPython的機能をLLVM IRで扱えるようにしています。

    - `visitors/`

      ASTノードを走査するためのクラス群。各構文要素（式、文、パターンなど）に応じた処理を振り分け、IR生成の実装を分割しています。

  - `pyc/compiler/`

    `ll2bin.py`や`ll2s.py`といったコンパイル補助スクリプトを提供し、生成された`.ll`(LLVM IR)をアセンブリやバイナリへ変換する機能をまとめています。

- **トップレベルのファイル構成**

  - `.gitignore`, `.python-version`, `.vscode/settings.json`

    開発環境・バージョン管理に関わる設定ファイルです。

  - `fib.py`, `fib.py.ll`, `fib.py.s`

    Fibonacci数列を例にしたサンプルコード（Python原本・生成したLLVM IR・アセンブリ）。

  - `fib.c`, `fib.c.ll`

    Fibonacci数列を例にしたサンプルコード（C原本・生成したLLVM IR）。

  - `helloworld.ll`

    “Hello, world!”のLLVM IRサンプル。

  - `sample.c`, `sample.ll`

    Cで書いたサンプルと、そのLLVM IR。

  - `source.py`, `source.py.ll`, `source.py.s`

    “Hello, world!”やFibonacciを含む実装のサンプルコード（Python原本・生成したLLVM IR・アセンブリ）。

  - `pyproject.toml`, `uv.lock`

    Poetryや依存管理の設定ファイル。`uv.lock`はバイナリ再現性を高めるためのロックファイルとして扱われています。

---

### 主な処理の流れ

1. **AST生成**

   `pyc/codegen/codegen.py`内でPythonのソースファイルを読み込み、`ast.parse`によりASTを構築します。

2. **LLVM IR生成**

   生成したASTを`pyc/codegen/generator.py`の`IRGenerator`や`visitors/`配下のクラスに渡し、各構文要素に応じて`IRBuilder`がLLVM IRを組み立てます。  
   結果として、`.ll`ファイルが出力されます。

3. **コンパイル**

   `pyc/compiler/ll2bin.py`や`ll2s.py`などにより、LLVM IR（`.ll`ファイル）をアセンブリ（`.s`ファイル）やネイティブバイナリへ変換します。環境によっては`llc`や`clang`を使用します。

4. **stdlibのサポート**

   `pyc/codegen/stdlib/`以下では、Pythonオブジェクトに相当する`PyInt`, `PyString`, `PyObject`などのC実装が含まれます。算術演算や文字列処理、例外ハンドリング、`print`などのPython的な機能をLLVM IRレベルで再現するための仕組みとなっています。
