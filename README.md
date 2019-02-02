# stark-rust-sample

[osuketh/stark\-rust: zk\-STARK for fibonacci sequence in Rust](https://github.com/osuketh/stark-rust)を触ってみる。
zk-STARKsとかよくわかっていないので実際に動くものを触って理解を深めたい。

## 準備

レポジトリの構成が一通り必要そうなので、`git clone`する。

```sh
❯ git clone https://github.com/osuketh/stark-rust
```

## 実行

```sh
❯ cargo run 52 9
   Compiling libstark_rs v0.1.0 (/Users/respepic/.go/src/github.com/cipepser/stark-rust)
warning: redundant linker flag specified for library `c++`

    Finished dev [unoptimized + debuginfo] target(s) in 44.70s
     Running `target/debug/libstark_rs 52 9`
verify:1
------------------------------------------------------------
| BAIR Specifications                                      |
------------------------------------------------------------
| field size                                     = 2^64    |
| number of variables per state (w)              = 3       |
| number of regular constraints (s)              = 1       |
| number of permutation constraints              = 0       |
| number of cycles (c)                           = (2^2)-1 |
| degree of constraint system (d)                = 1       |
| degree of permutation constraint system        = -1      |
| number of boundary constraints (B)             = 3       |
| number of variables used by constraint systems = 3       |
| number of variables routed                     = 0       |
| number of variables not routed                 = 3       |
------------------------------------------------------------
-------------------------------------------------
| ACSP Specifications                           |
-------------------------------------------------
| field size                             = 2^64 |
| number of algebraic-registers (|\Tau|) = 3    |
| number of neighbors (|N|)              = 9    |
| vanishing space size                   = 2^3  |
| composition degree bound               = 15   |
-------------------------------------------------
-------------------------------------------------------------
| APR Specifications                                        |
-------------------------------------------------------------
| field size                                       = 2^64   |
| number of algebraic-registers (|\Tau|)           = 3      |
| number of neighbors (|N|)                        = 9      |
| witness (f) evaluation space size (|L|)          = 2^8    |
| constraint (g) evaluation space size (|L_{cmp}|) = 2^6    |
| witness (f) maximal rate (\rho_{max})            = 2^{-5} |
| constraint (g) rate (\rho_{cmp})                 = 2^{-3} |
| zero knowledge parameter (k)                     = 1      |
| rate parameter (R)                               = 3      |
| constraints degree log (d)                       = 3      |
-------------------------------------------------------------
Constructing APR (ACSP) witness:.(0.0004 seconds)
-----------------------------------------
| FRI for witness (f) specifications #1 |
-----------------------------------------
| field size (|F|)      = 2^64          |
| RS code dimension     = 2^3           |
| RS block-length       = 2^8           |
| RS code rate          = 2^-{5}        |
| Soundness error       = 2^-{61}       |
| dim L_0 (eta)         = 2             |
| recursion depth       = 1             |
| COMMIT repetitions    = 2             |
| number of tests (ell) = 13            |
-----------------------------------------
---------------------------------------------
| FRI for constraints (g) specifications #1 |
---------------------------------------------
| field size (|F|)      = 2^64              |
| RS code dimension     = 2^3               |
| RS block-length       = 2^6               |
| RS code rate          = 2^-{3}            |
| Soundness error       = 2^-{61}           |
| dim L_0 (eta)         = 2                 |
| recursion depth       = 1                 |
| COMMIT repetitions    = 2                 |
| number of tests (ell) = 21                |
---------------------------------------------
communication iteration #1:.(0.000488 seconds)
communication iteration #2:.(0.000778 seconds)
communication iteration #3:.(0.000773 seconds)
communication iteration #4:.(0.000806 seconds)
communication iteration #5:.(0.001798 seconds)
Verifier decision: ACCEPT
------------------------------------------------------------------------
| Protocol execution measurements                                      |
------------------------------------------------------------------------
| Prover time                                       = 0.002945 Seconds |
| Verifier time                                     = 0.003251 Seconds |
| Total IOP length                                  = 28.011719 KBytes |
| Total communication complexity (STARK proof size) = 7.140625 KBytes  |
| Query complexity                                  = 5.640625 KBytes  |
------------------------------------------------------------------------
```

### `error: header 'libSTARK/fsrs/Fsrs_wrapper.hpp' does not exist.`が出る

準備不足？

別フォルダでやってだめだった。

```sh
❯ git clone git@github.com:LayerXcom/libSTARK.git
❯ cd libSTARK
❯ make -j8
```

だめなエラー

`ld: symbol(s) not found for architecture x86_64`

osukeさんのrepoを見るとrepo直下に`libSTARK`があるので、`stark-rust`直下で`git clone`してみる。

```sh
❯ git clone https://github.com/LayerXcom/libSTARK
```


```sh
❯ cargo run 52 9
```

同じエラーでだめ。

```sh
--- stderr
error: header 'libSTARK/fsrs/Fsrs_wrapper.hpp' does not exist.
thread 'main' panicked at 'Unable to generte bindings: ()', src/libcore/result.rs:1009:5
note: Run with `RUST_BACKTRACE=1` for a backtrace.

`libstark_rs`のbuildで失敗する。

```

`make`してみる。

```sh
❯ cd libSTARK
❯ make -j8
```

当然のように失敗する。

```sh
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make[1]: *** [/Users/respepic/.go/src/github.com/cipepser/stark-rust/libSTARK/bin/starkdpm/stark-dpm] Error 1
make: *** [stark-dpm] Error 2
```

`libSTARK/fsrs/Fsrs_wrapper.hpp`がないと言っているので、確認したら`libSTARK/fsrs`がない。。。

```sh
# in libSTARK
❯ git checkout 9c45
❯ make -j8
```

```sh
# in stark-rust
❯ cargo run 52 9
```

やっと動いた。

## 理解する

別の数字で試してみる。

`src/lib.rs`にあるとおり、`A`と`B`を与えるので、ここをいじってみる。

```rust
#[structopt(name = "A: initial num")]
input_a: c_uint,
#[structopt(name = "B: initial num")]
input_b: c_uint,
```


```sh
❯ cargo run 2 3
    Finished dev [unoptimized + debuginfo] target(s) in 0.16s
     Running `target/debug/libstark_rs 2 3`
verify:0
------------------------------------------------------------
| BAIR Specifications                                      |
------------------------------------------------------------
| field size                                     = 2^64    |
| number of variables per state (w)              = 3       |
| number of regular constraints (s)              = 1       |
| number of permutation constraints              = 0       |
| number of cycles (c)                           = (2^2)-1 |
| degree of constraint system (d)                = 1       |
| degree of permutation constraint system        = -1      |
| number of boundary constraints (B)             = 3       |
| number of variables used by constraint systems = 3       |
| number of variables routed                     = 0       |
| number of variables not routed                 = 3       |
------------------------------------------------------------
-------------------------------------------------
| ACSP Specifications                           |
-------------------------------------------------
| field size                             = 2^64 |
| number of algebraic-registers (|\Tau|) = 3    |
| number of neighbors (|N|)              = 9    |
| vanishing space size                   = 2^3  |
| composition degree bound               = 15   |
-------------------------------------------------
-------------------------------------------------------------
| APR Specifications                                        |
-------------------------------------------------------------
| field size                                       = 2^64   |
| number of algebraic-registers (|\Tau|)           = 3      |
| number of neighbors (|N|)                        = 9      |
| witness (f) evaluation space size (|L|)          = 2^8    |
| constraint (g) evaluation space size (|L_{cmp}|) = 2^6    |
| witness (f) maximal rate (\rho_{max})            = 2^{-5} |
| constraint (g) rate (\rho_{cmp})                 = 2^{-3} |
| zero knowledge parameter (k)                     = 1      |
| rate parameter (R)                               = 3      |
| constraints degree log (d)                       = 3      |
-------------------------------------------------------------
Constructing APR (ACSP) witness:.(0.000283 seconds)
-----------------------------------------
| FRI for witness (f) specifications #1 |
-----------------------------------------
| field size (|F|)      = 2^64          |
| RS code dimension     = 2^3           |
| RS block-length       = 2^8           |
| RS code rate          = 2^-{5}        |
| Soundness error       = 2^-{61}       |
| dim L_0 (eta)         = 2             |
| recursion depth       = 1             |
| COMMIT repetitions    = 2             |
| number of tests (ell) = 13            |
-----------------------------------------
---------------------------------------------
| FRI for constraints (g) specifications #1 |
---------------------------------------------
| field size (|F|)      = 2^64              |
| RS code dimension     = 2^3               |
| RS block-length       = 2^6               |
| RS code rate          = 2^-{3}            |
| Soundness error       = 2^-{61}           |
| dim L_0 (eta)         = 2                 |
| recursion depth       = 1                 |
| COMMIT repetitions    = 2                 |
| number of tests (ell) = 21                |
---------------------------------------------
communication iteration #1:.(0.000802 seconds)
communication iteration #2:.(0.001036 seconds)
communication iteration #3:.(0.000354 seconds)
communication iteration #4:(0.000392 seconds)
communication iteration #5:.(0.001405 seconds)
Verifier decision: REJECT
------------------------------------------------------------------------
| Protocol execution measurements                                      |
------------------------------------------------------------------------
| Prover time                                       = 0.002989 Seconds |
| Verifier time                                     = 0.002219 Seconds |
| Total IOP length                                  = 28.011719 KBytes |
| Total communication complexity (STARK proof size) = 7.140625 KBytes  |
| Query complexity                                  = 5.640625 KBytes  |
------------------------------------------------------------------------
```

[差分](https://docs.google.com/spreadsheets/d/1U9fXsWUlzJwm9qI5laFC_PBJSqin5FXS9xBPbJD8yNE/edit?usp=sharing)を確認。
計算時間を除くと、差分があったのは結果のところのみ。

```sh
❯ cargo run 2 3				❯ cargo run 52 9
verify:0				verify:1
Verifier decision: REJECT				Verifier decision: ACCEPT
```

[How to run the code](https://github.com/osuketh/stark-rust#how-to-run-the-code)にも以下のように書いてある

> The above execution results in execution of STARK simulation over the fibonacchi sequence.The statement is "I know secret numbers A, B such that the 5th element of the Fibonacci sequence is 131. They are A=52, B=9"

フィボナッチ数列の5番目の要素(`131`)を知っているかをzkしている。

`52, 9, 61, 70, 131`

なので、正しい。

### 別の数字でも`ACCEPT`はず？？

```
a4 = 131

a0, a1, a2,    a3,    a4
a0, a1, a0+a1, a1+a2, a2+a3

a2 = a0 + a1
a3 = a1 + a2

a4 = a2 + a3
   = a2 + (a1 + a2)
   = 2 * a2 + a1
   = 2 * (a0 + a1) + a1
   = 2*a0 + 3*a1
```

上記より`2x + 3y = 131`
となるような`(x, y)`の組を与えれば`ACCEPT`されるはず。

`2x`は偶数なので、`3y`を奇数になるようにすると

`(x, y) = (64, 1), (61, 3), (58, 5), (55, 7), (52, 9), (49, 11), ..., (1, 43)`

※ 二元一次不定方程式`2x + 3y = 131`の解`(x, y)`は負数も取りうるが、今回は`c_uint`で型を付けているので対象外とした。
※ ちなみに特殊解`(x, y) = (52, 9)`を知っているので、一般解は整数`m`を用いて`(x, y) = (52 + 3m, 9 - 2m)`となる。

`(52, 9)`も含まれていることがわかる。試してみよう。

```sh
❯ cargo run 64 1
verify:0
```

あれ、だめだった。
`64, 1, 65, 66, 131`なはずでは？
他はも試してみよう。

```sh
❯ cargo run 1 43
verify:0

❯ cargo run 49 11
verify:0
```

`(52, 9)`じゃないとだめっぽい。
[fibonacciの実装](https://github.com/LayerXcom/libSTARK/blob/libstark-rs/fibonacchiseq/main.cpp)をみていく必要がありそう。


## References
* [osuketh/stark\-rust: zk\-STARK for fibonacci sequence in Rust](https://github.com/osuketh/stark-rust)
* [git clone git@github\.com:LayerXcom/libSTARK\.git](https://github.com/LayerXcom/libSTARK/tree/libstark-rs/fsrs)
