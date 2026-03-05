# 第8章 制御構文と関数

---

## 目次

1. [制御構文：if 文](#1-制御構文if-文)
2. [制御構文：for 文](#2-制御構文for-文)
3. [制御構文：while 文](#3-制御構文while-文)
4. [関数](#4-関数)

---

## 1. 制御構文：if 文

### 基本構文

```python
if 条件:
    条件がTrueのときの処理
elif 別の条件:
    別の条件がTrueのときの処理
else:
    どの条件にも当てはまらないときの処理
```

> ⚠️ **インデント**: Python では **インデント（字下げ）** でブロックの範囲を表します。通常は**スペース4つ**を使います。インデントが揃っていないとエラーになります。

### 例：GC含量の判定

```python
gc_content = 0.65

if gc_content > 0.6:
    print("High GC content")
elif gc_content > 0.4:
    print("Medium GC content")
else:
    print("Low GC content")
```

```
High GC content
```

### 例：コドンの判定

```python
codon = "TAA"

if codon == "ATG":
    print(f"{codon} is a start codon")
elif codon in ["TAA", "TAG", "TGA"]:
    print(f"{codon} is a stop codon")
else:
    print(f"{codon} is a coding codon")
```

```
TAA is a stop codon
```

---

## 2. 制御構文：for 文

### 基本構文

```python
for 変数 in イテラブル:
    処理
```

**イテラブル**とは、繰り返し処理できるオブジェクト（リスト、文字列、`range()` など）のことです。

### リストのループ

```python
genes = ["mnmG", "gyrB", "dnaN"]

for gene in genes:
    print(f"Processing {gene}...")
```

```
Processing mnmG...
Processing gyrB...
Processing dnaN...
```

### range() を使ったループ

```python
# range(5) → 0, 1, 2, 3, 4
for i in range(5):
    print(i)

# range(1, 6) → 1, 2, 3, 4, 5
for i in range(1, 6):
    print(i)

# range(0, 10, 2) → 0, 2, 4, 6, 8（2刻み）
for i in range(0, 10, 2):
    print(i)
```

### 文字列のループ

```python
seq = "ATGC"

for base in seq:
    print(base)
```

```
A
T
G
C
```

### 辞書のループ

```python
codon_table = {"ATG": "Met", "GCT": "Ala", "TAA": "Stop"}

for codon, amino_acid in codon_table.items():
    print(f"{codon} -> {amino_acid}")
```

```
ATG -> Met
GCT -> Ala
TAA -> Stop
```

### 実用例：塩基の出現回数をカウント

```python
seq = "ATGCGATCGATCGATCG"

for base in ["A", "T", "G", "C"]:
    count = seq.count(base)
    print(f"{base}: {count}")
```

```
A: 4
T: 4
G: 5
C: 4
```

### break と continue

```python
# break：条件を満たしたらループを終了
genes = ["mnmG", "gyrB", "dnaN", "atpA"]

for gene in genes:
    if gene == "dnaN":
        print(f"{gene} が見つかりました")
        break
    print(f"{gene} はスキップ")
```

```
mnmG はスキップ
gyrB はスキップ
dnaN が見つかりました
```

```python
# continue：その回だけスキップして次へ
lines = ["# comment", "mnmG\t1881", "# comment", "gyrB\t2400"]

for line in lines:
    if line.startswith("#"):
        continue    # コメント行はスキップ
    print(line)
```

```
mnmG	1881
gyrB	2400
```

### リスト内包表記

`for` 文を使ったリスト作成を1行で書く構文です。

```python
# 0〜9 の二乗のリスト
squares = [x**2 for x in range(10)]
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# ATP合成酵素遺伝子だけ抽出（if と組み合わせる）
genes = ["mnmG", "gyrB", "dnaN", "atpA", "leuA", "trpG"]
atp_genes = [g for g in genes if g.startswith("atp")]
# ["atpA"]
```

通常の `for` 文で書くと次のようになります。リスト内包表記はその短縮形です。

```python
atp_genes = []
for g in genes:
    if g.startswith("atp"):
        atp_genes.append(g)
```

---

## 3. 制御構文：while 文

`while` 文は、条件が `True` の間、処理を繰り返します。繰り返す回数が決まっていないときに使います。

```python
count = 0

while count < 5:
    print(f"count = {count}")
    count += 1    # count = count + 1 と同じ
```

```
count = 0
count = 1
count = 2
count = 3
count = 4
```

> ⚠️ **注意**: 条件が永遠に `True` のままだと**無限ループ**になります。JupyterLab で無限ループに入ってしまった場合は、上部メニューの **Kernel → Interrupt Kernel**（または停止ボタン ■）で中断できます。

---

## 4. 関数

### 関数とは

**関数**は、一連の処理をまとめて名前を付けたものです。同じ処理を何度も使い回すときに便利です。

### 関数の定義

```python
def 関数名(引数1, 引数2, ...):
    処理
    return 戻り値
```

### 例1：GC含量を計算する関数

```python
def calc_gc_content(sequence):
    """DNA配列のGC含量を計算する"""
    g_count = sequence.count("G")
    c_count = sequence.count("C")
    gc_content = (g_count + c_count) / len(sequence)
    return gc_content

seq = "ATGCGATCGATCG"
result = calc_gc_content(seq)
print(f"GC content: {result:.3f}")    # GC content: 0.538
```

> **`"""..."""` とは**: 関数の直下に書く文字列は **docstring**（ドキュメント文字列）と呼ばれ、関数の説明を記述します。

### 例2：相補鎖を返す関数

```python
def complement(sequence):
    """DNA配列の相補鎖を返す"""
    table = str.maketrans("ATGC", "TACG")
    return sequence.translate(table)

def reverse_complement(sequence):
    """DNA配列の逆相補鎖を返す"""
    return complement(sequence)[::-1]

seq = "ATGCGATCGA"
print(f"Original:            {seq}")
print(f"Complement:          {complement(seq)}")
print(f"Reverse complement:  {reverse_complement(seq)}")
```

```
Original:            ATGCGATCGA
Complement:          TACGCTAGCT
Reverse complement:  TCGATCGCAT
```

> **`[::-1]` とは**: スライスの応用で、文字列やリストを逆順にします。

### デフォルト引数

引数にデフォルト値を設定できます。呼び出し時に省略するとデフォルト値が使われます。

```python
def calc_gc_content(sequence, round_digits=3):
    """DNA配列のGC含量を計算する"""
    gc = (sequence.count("G") + sequence.count("C")) / len(sequence)
    return round(gc, round_digits)

print(calc_gc_content("ATGCGATCG"))        # 0.556（デフォルト: 小数3桁）
print(calc_gc_content("ATGCGATCG", 1))     # 0.6（小数1桁）
```

---

## まとめ

| 構文 | 用途 |
|------|------|
| `if / elif / else` | 条件によって処理を分ける |
| `for` | リスト・文字列・辞書などを順に処理する |
| `while` | 条件が満たされる間、繰り返す |
| `break` | ループを途中で終了する |
| `continue` | その回の処理をスキップして次へ |
| `def` | 処理をまとめて関数として定義する |
| `return` | 関数の戻り値を指定する |
