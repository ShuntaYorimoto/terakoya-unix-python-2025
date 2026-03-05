# 第6章 Python入門：数値・文字列・変数

---

## 目次

1. [Python とは](#1-python-とは)
2. [変数とデータ型](#2-変数とデータ型)
3. [演算子](#3-演算子)
4. [文字列の操作](#4-文字列の操作)

---

## 1. Python とは

### 1-1. なぜプログラミングを学ぶのか

- **自由度**: データの前処理や解析、可視化などさまざまな用途に使用できます。既存のソフトウェアでは手の届かないことも、自身でカスタマイズして実行することができます。
- **再現性**: コードは「解析の記録」になります。「どのように処理したか」が誰でも確認・再現できる形で残ります。
- **自動化**: 一度コードを作成すれば似た用途に転用でき、時間の節約に繋がります。

### 1-2. なぜ Python なのか

#### 読みやすい文法

プログラミング言語にはさまざまな種類があります。「DNA 配列の GC 含量を計算する」という同じタスクを、いくつかの言語で書き比べてみましょう。

**C 言語**:

```c
#include <stdio.h>
#include <string.h>

int main() {
    char seq[] = "ATGCGATCGA";
    int len = strlen(seq);
    int gc = 0;
    for (int i = 0; i < len; i++) {
        if (seq[i] == 'G' || seq[i] == 'C') gc++;
    }
    printf("GC content: %.2f\n", (float)gc / len);
    return 0;
}
```

**Java**:

```java
public class GCContent {
    public static void main(String[] args) {
        String seq = "ATGCGATCGA";
        long gc = seq.chars()
                     .filter(c -> c == 'G' || c == 'C')
                     .count();
        System.out.printf("GC content: %.2f%n", (double) gc / seq.length());
    }
}
```

**R**:

```r
seq <- "ATGCGATCGA"
chars <- strsplit(seq, "")[[1]]
gc_content <- sum(chars %in% c("G", "C")) / nchar(seq)
cat(sprintf("GC content: %.2f\n", gc_content))
```

**Python**:

```python
seq = "ATGCGATCGA"
gc_content = (seq.count("G") + seq.count("C")) / len(seq)
print(f"GC content: {gc_content:.2f}")
```

Pythonは可読性の高い言語として設計されており、簡潔にコードを実装することが可能です。そのため、プログラミング初心者にとっても習得しやすい言語の一つになります。

#### 豊富なライブラリ

さまざまな用途に対応したライブラリが豊富に存在します。本講義で取り上げるのはそれらのほんの一部です。

- **NumPy**: 高速な数値計算・配列操作
- **Pandas**: 表形式データの読み込み・集計・フィルタリング
- **Matplotlib / Seaborn**: グラフ・ヒートマップなどの可視化
- **BioPython**: バイオインフォマティクスのデータフォーマットに対応するファイルの読み書きなど

#### 大きなコミュニティ

多くの研究者やエンジニア、データサイエンティストが利用しているプログラミング言語となっています。ウェブや書籍など情報源が豊富で、困ったときに解決策を見つけやすい環境が整っています。

#### R との使い分け

バイオインフォマティクス分野では、目的に応じて主に R と Python が使われます。R は統計解析や RNA-seq 解析などの NGS 解析に強い印象です。一方、Python は汎用スクリプト・機械学習・Web 連携など幅広い用途で使われます。どちらが「正解」ということはなく、結局両方使うことになります。本講座では、 Python を扱います。

### 1-3. 最初の一歩

Code セルに以下を入力して `Shift + Enter` で実行してみましょう。

```python
print("Hello, Bioinformatics!")
```

```
Hello, Bioinformatics!
```

> **`print()` 関数**: 括弧内の値を画面に出力します。JupyterLab ではセルの最後の式は `print()` なしでも表示されますが、途中の結果を確認したいときは明示的に `print()` を使います。

---

## 2. 変数とデータ型

### 変数とは

**変数**は、値に名前を付けて保存する仕組みです。`=` を使って値を代入します。

```python
sequence = "ATGCGATCGA"
length = 10
gc_content = 0.5
```

> **命名規則**: Python では変数名に**小文字とアンダースコア**を使うのが慣例です（例：`gene_name`, `seq_length`）。数字で始めることはできません。また、Pythonの予約語（True, if, andなど）も変数名として使うことはできません。

### 基本的なデータ型

| 型 | 名前 | 例 |
|----|------|-----|
| `int` | 整数 | `42`, `1000` |
| `float` | 浮動小数点数 | `3.14`, `1e-10` |
| `str` | 文字列 | `"ATGC"`, `'gene_001'` |
| `bool` | 真偽値 | `True`, `False` |

```python
# 整数（int）
seq_length = 10
num_genes = 20000

# 浮動小数点数（float）
gc_content = 0.6
evalue = 1e-50

# 文字列（str）
gene_name = "eyeless"
sequence = "ATGCGATCGA"

# 真偽値（bool）
is_coding = True
```

### 型の確認と変換

`type()` 関数で変数の型を確認できます。

```python
print(type(gene_name))    # <class 'str'>
print(type(seq_length))   # <class 'int'>
print(type(gc_content))   # <class 'float'>
print(type(is_coding))    # <class 'bool'>
```

異なる型に変換したいときは、`int()`, `float()`, `str()` を使います。

```python
# 文字列 → 整数
num = int("42")        # 42

# 整数 → 文字列
text = str(100)        # "100"

# 文字列 → 浮動小数点数
val = float("3.14")    # 3.14
```

---

## 3. 演算子

### 算術演算子

```python
a = 10
b = 3

print(a + b)     # 13    加算
print(a - b)     # 7     減算
print(a * b)     # 30    乗算
print(a / b)     # 3.333... 除算（結果は float）
print(a // b)    # 3     整数除算（切り捨て）
print(a % b)     # 1     剰余（あまり）
print(a ** b)    # 1000  べき乗
```

### 比較演算子

比較の結果は `True` または `False` になります。

```python
x = 5
print(x == 5)    # True   等しい
print(x != 3)    # True   等しくない
print(x > 3)     # True   より大きい
print(x >= 5)    # True   以上
print(x < 10)    # True   より小さい
print(x <= 4)    # False  以下
```

### 論理演算子

複数の条件を組み合わせるときに使います。

```python
x = 5
print(x > 0 and x < 10)    # True   両方True → True
print(x > 0 or x > 100)    # True   どちらかTrue → True
print(not x > 10)           # True   Trueを反転
```

---

## 4. 文字列の操作

バイオインフォマティクスでは配列データを文字列として扱うことが多いため、文字列操作は重要です。

### メソッドとは

Pythonでは、変数（オブジェクト）に対して `.メソッド名()` の形で呼び出せる**専用の関数**があります。これを**メソッド**と呼びます。

```python
species = "ceratovacuna jaonica"
species.capitalize()   # species という文字列オブジェクトの capitalize メソッドを呼び出す
```

`len(seq)` のような**組み込み関数**は `関数名(対象)` の形で呼びますが、メソッドは `対象.メソッド名()` の形で呼ぶのが特徴です。

### 基本操作

```python
seq = "ATGCGATCGA"

# 長さ
print(len(seq))          # 10

# 連結
seq2 = seq + "TTT"       # "ATGCGATCGATTT"

# 繰り返し
poly_a = "A" * 10        # "AAAAAAAAAA"

# 大文字・小文字
print(seq.lower())       # "atgcgatcga"
print("atgc".upper())    # "ATGC"
```

### インデックスとスライス

文字列の各文字には**0から始まる番号**（インデックス）が付いています。

```
 A  T  G  C  G  A  T  C  G  A
 0  1  2  3  4  5  6  7  8  9
-10-9 -8 -7 -6 -5 -4 -3 -2 -1
```

```python
seq = "ATGCGATCGA"

# インデックスで1文字取得（0始まり）
print(seq[0])       # "A"（最初の文字）
print(seq[3])       # "C"（4番目の文字）
print(seq[-1])      # "A"（最後の文字）

# スライスで部分文字列を取得 [開始:終了]
# ※ 終了位置の文字は含まれない
print(seq[0:3])     # "ATG"（開始コドン）
print(seq[3:6])     # "CGA"
print(seq[:3])      # "ATG"（先頭から3文字）
print(seq[-3:])     # "CGA"（末尾3文字）
```

> ⚠️ **注意**: インデックスは **0 から始まる**ことに注意してください。`seq[1]` は2番目の文字です。

### よく使うメソッド

```python
seq = "ATGCGATCGA"

# 文字数のカウント
print(seq.count("G"))        # 3（G の出現回数）
print(seq.count("AT"))       # 2（AT の出現回数）

# 検索（見つかった位置を返す。見つからなければ -1）
print(seq.find("GAT"))       # 4
print(seq.find("NNN"))       # -1

# 置換
print(seq.replace("T", "U")) # "AUGCGAUCGA"（DNA → RNA）

# 特定の文字列で始まる/終わるか
print(seq.startswith("ATG")) # True
print(seq.endswith("TGA"))   # False

# 分割
header = "sp|P26367|PAX6_HUMAN"
parts = header.split("|")    # ["sp", "P26367", "PAX6_HUMAN"]
print(parts[1])              # "P26367"

# 前後の空白・改行を除去
line = "  ATGCGATCGA\n"
print(line.strip())          # "ATGCGATCGA"（ファイルを1行ずつ読むときに便利）
```

### f文字列（フォーマット済み文字列）

変数の値を文字列に埋め込むには **f文字列**（f-string）が便利です。

```python
gene = "mnmG"
length = 1881
print(f"{gene} は {length} bp")
# mnmG は 1881 bp

gc = 0.4567
print(f"GC content: {gc:.2f}")
# GC content: 0.46（小数点以下2桁に丸め）
```

---

## まとめ

| カテゴリ | 要素 | 概要 |
|---------|------|------|
| データ型 | `int`, `float`, `str`, `bool` | 整数、小数、文字列、真偽値 |
| 演算子 | `+`, `-`, `*`, `/`, `//`, `%`, `**` | 算術演算 |
| 演算子 | `==`, `!=`, `>`, `<`, `>=`, `<=` | 比較演算 |
| 演算子 | `and`, `or`, `not` | 論理演算 |
| 文字列操作 | インデックス・スライス・メソッド | 配列データの処理 |
| f文字列 | `f"{変数}"` | 変数の埋め込み |
