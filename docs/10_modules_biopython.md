# 第10章 モジュール・BioPython

---

## 目次

1. [モジュールのインポート](#1-モジュールのインポート)
2. [sys](#2-sys)
3. [正規表現](#3-正規表現)
4. [BioPython](#4-biopython)

---

## 1. モジュールのインポート

### モジュールとは

**モジュール**は、関数や変数をまとめたPythonファイルです。`import` 文で読み込んで利用します。

### import の書き方

```python
# モジュール全体をインポート
import math
print(math.sqrt(16))    # 4.0

# 特定の関数だけインポート
from math import sqrt, pi
print(sqrt(16))          # 4.0

# 別名を付けてインポート（慣例的な略称がある）
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

---

## 2. sys

### sys：スクリプトの引数と実行環境

JupyterLab よりも、**コマンドラインスクリプト**として実行するときに使います。

```python
import sys

# Python のバージョン確認
print(sys.version)

# コマンドライン引数の取得
# スクリプトを "python myscript.py input.fasta output.txt" のように実行した場合
# sys.argv[0] = "myscript.py"
# sys.argv[1] = "input.fasta"
# sys.argv[2] = "output.txt"

input_file = sys.argv[1]
output_file = sys.argv[2]
```

#### スクリプトとして実行する

Python ファイルをコマンドラインから直接実行する場合、慣例として以下のパターンを使います。

myscript.py
```python
#!/usr/bin/env python3

import sys

def main():
    input_file = sys.argv[1]
    output_file = sys.argv[2]
    print(f"{input_file} を処理して {output_file} に書き出します")

if __name__ == "__main__":
    main()
```

> **シバン（`#!`）とは**: スクリプトの1行目に書く特殊なコメントで、どのインタープリタで実行するかを OS に伝えます。
> `#!/usr/bin/env python3` は `$PATH` から `python3` を探すため、環境を問わず動作します。
> `chmod +x myscript.py` で実行権限を付与すると、`python3 myscript.py` の代わりに
> `./myscript.py` と直接実行できます。

`if __name__ == "__main__":` のブロックは、**スクリプトとして直接実行されたときだけ**実行されます。別のファイルから `import` されたときは実行されません。

```bash
# コマンドラインから実行
python3 myscript.py input.fasta output.txt
```

> **Jupyter vs スクリプト**: 探索的なデータ解析には JupyterLab が向いています。一方、同じ処理を多数のファイルに繰り返す自動化・バッチ処理には、コマンドライン引数を受け取るスクリプト形式が便利です。

---

## 3. 正規表現

**正規表現**（Regular Expression / regex）は、文字列のパターンを表す記法です。`re` モジュールで使います。

```python
import re
```

### よく使うパターン記号

| 記号 | 意味 | 例 |
|------|------|-----|
| `.` | 任意の1文字 | `A.G` → "ATG", "ACG", "AAG" |
| `*` | 直前の文字が0回以上 | `AT*` → "A", "AT", "ATT" |
| `+` | 直前の文字が1回以上 | `AT+` → "AT", "ATT", "ATTT" |
| `?` | 直前の文字が0回または1回 | `AT?` → "A", "AT" |
| `^` | 行の先頭 | `^>` → `>` で始まる行 |
| `$` | 行の末尾 | `\n$` |
| `[...]` | いずれかの1文字 | `[ATGC]` → A, T, G, C のどれか |
| `\d` | 数字1文字 | `\d+` → 1つ以上の数字 |
| `\s` | 空白文字（スペース・タブ・改行） | |

### 主な関数

```python
import re

text = "sp|P26367|PAX6_HUMAN Paired box protein Pax-6"

# パターンに一致する最初の箇所を検索
match = re.search(r"P\d{5}", text)    # P + 数字5桁
if match:
    print(match.group())    # P26367

# パターンに一致する全箇所をリストで返す
ids = re.findall(r"[A-Z0-9]+_[A-Z]+", text)
print(ids)    # ['PAX6_HUMAN']

# 置換
clean = re.sub(r"\s+", " ", "PAX6  \t  PAX5")    # 連続する空白を1つのスペースに
print(clean)    # "PAX6 PAX5"
```

### 実用例：FASTA ヘッダから UniProt ID を抽出

```python
import re

headers = [
    ">sp|P26367|PAX6_HUMAN Paired box protein Pax-6",
    ">sp|P49640|PAX4_HUMAN Paired box protein Pax-4",
    ">sp|P23759|PAX7_HUMAN Paired box protein Pax-7",
]

for header in headers:
    match = re.search(r"sp\|([A-Z0-9]+)\|", header)
    if match:
        uniprot_id = match.group(1)    # group(1) でキャプチャグループ1を取得
        print(uniprot_id)
```

```
P26367
P49640
P23759
```

> **`r"..."` とは**: 正規表現は `r` を文字列の前に付けた **raw 文字列**で書くのが慣例です。`\` をエスケープせずにそのまま書けます。

---

## 4. BioPython

BioPython は、バイオインフォマティクス解析のための包括的なライブラリです。FASTA/FASTQ の読み書き、配列操作、GC含量計算などが簡単に行えます。

### Seq オブジェクト

`Seq` オブジェクトは、塩基配列やアミノ酸配列を扱うための基本クラスです。文字列に似た操作に加え、生物学的な操作（相補鎖・翻訳など）が使えます。

```python
from Bio.Seq import Seq

dna = Seq("ATGCGATCGATCGATCG")

# 文字列と同様の操作
print(len(dna))           # 17
print(dna[0:3])           # ATG
print(dna.count("G"))     # 5

# 生物学的な操作
print(dna.complement())           # TACGCTAGCTAGCTAGC
print(dna.reverse_complement())   # CGATCGATCGATCGCAT
print(dna[:12].translate())       # MRSI（最初の4コドン）
```

### SeqRecord オブジェクト

`SeqRecord` は、配列（`Seq`）にID・説明文などのメタデータを付けたオブジェクトです。FASTAファイルの1エントリに対応します。

```python
from Bio.SeqRecord import SeqRecord
from Bio.Seq import Seq

record = SeqRecord(
    Seq("ATGCGATCGATCG"),
    id="gene_001",
    description="hypothetical protein BucCj_0090 [Buchnera aphidicola (Ceratovacuna japonica)]"
)

print(record.id)           # gene_001
print(record.description)  # hypothetical protein BucCj_0090 [Buchnera aphidicola (Ceratovacuna japonica)]
print(record.seq)          # ATGCGATCGATCG
print(len(record))         # 13
```

### SeqIO：FASTAファイルの読み込み

```python
from Bio import SeqIO

# FASTAファイルを読み込む（リストとして）
records = list(SeqIO.parse("pax_human.fasta", "fasta"))

print(f"配列数: {len(records)}")

for record in records:
    print(f"{record.id}: {len(record)} aa")
```

```
配列数: 4
sp|P26367|PAX6_HUMAN: 422 aa
sp|Q02548|PAX5_HUMAN: 391 aa
...
```

辞書形式で読み込むと、IDを使って配列を検索できます。

```python
# IDをキーとして辞書形式で読み込む
seq_dict = SeqIO.to_dict(SeqIO.parse("pax_human.fasta", "fasta"))

pax6 = seq_dict["sp|P26367|PAX6_HUMAN"]
print(pax6.seq[:50])
```

### SeqIO：FASTAファイルへの書き出し

```python
from Bio.SeqRecord import SeqRecord
from Bio.Seq import Seq
from Bio import SeqIO

new_records = [
    SeqRecord(Seq("ATGCGATCG"), id="seq_001", description=""),
    SeqRecord(Seq("TTGCGATCG"), id="seq_002", description=""),
]

with open("output.fasta", "w") as f:
    SeqIO.write(new_records, f, "fasta")
```

### 実用例：短い配列だけ抽出

```python
from Bio import SeqIO

input_file = "pax_human.fasta"
output_file = "short_pax.fasta"
max_length = 425

short_records = [
    record for record in SeqIO.parse(input_file, "fasta")
    if len(record) <= max_length
]

print(f"{len(short_records)} 配列が {max_length} aa 以下")

with open(output_file, "w") as f:
    SeqIO.write(short_records, f, "fasta")
```

> **複数行のリスト内包表記**: `[...]` の中では改行が自由に使えます（暗黙の行継続）。
> `for` 句と `if` 句を別の行に書くと、長い処理でも読みやすくなります。

### SeqUtils：GC含量の計算

```python
from Bio import SeqUtils, SeqIO

for record in SeqIO.parse("genome.fasta", "fasta"):
    gc = SeqUtils.gc_fraction(record.seq) * 100
    print(f"{record.id}: GC = {gc:.1f}%")
```

```
contig1: GC = 43.2%
contig2: GC = 43.8%
...
```

---

## まとめ

| モジュール | 主な用途 | よく使う関数・クラス |
|-----------|---------|-------------------|
| `sys` | コマンドライン引数、実行環境 | `sys.argv`, `sys.version` |
| `re` | 正規表現によるパターンマッチング | `search()`, `findall()`, `sub()` |
| `Bio.Seq` | 塩基/アミノ酸配列の操作 | `Seq`, `complement()`, `translate()` |
| `Bio.SeqRecord` | 配列+メタデータ | `SeqRecord` |
| `Bio.SeqIO` | FASTA/FASTQの読み書き | `parse()`, `write()`, `to_dict()` |
| `Bio.SeqUtils` | 配列の統計 | `gc_fraction()` |
