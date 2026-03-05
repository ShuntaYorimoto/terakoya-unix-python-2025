# 第7章 データ構造：リスト・タプル・辞書・集合

---

## 目次

1. [リスト](#1-リスト)
2. [タプル](#2-タプル)
3. [辞書](#3-辞書)
4. [集合](#4-集合)

---

## 1. リスト

### リストとは

**リスト**（`list`）は、複数の値を順番に格納するデータ型です。`[]` で作成します。

```python
# 遺伝子名のリスト
genes = ["mnmG", "gyrB", "dnaN", "atpA"]

# 数値のリスト
scores = [98.5, 75.2, 82.3, 91.0]

# 空のリスト
results = []
```

### 基本操作

```python
genes = ["mnmG", "gyrB", "dnaN", "atpA"]

# 要素数
print(len(genes))        # 4

# インデックスでアクセス（0始まり）
print(genes[0])          # "mnmG"
print(genes[-1])         # "atpA"

# スライス
print(genes[1:3])        # ["gyrB", "dnaN"]

# 要素の変更
genes[0] = "leuA"        # ["leuA", "gyrB", "dnaN", "atpA"]

# 要素の存在確認
print("gyrB" in genes)   # True
```

### 要素の追加・削除

```python
genes = ["mnmG", "gyrB"]

# 末尾に追加
genes.append("dnaN")          # ["mnmG", "gyrB", "dnaN"]

# 指定位置に挿入
genes.insert(1, "atpA")       # ["mnmG", "atpA", "gyrB", "dnaN"]

# 要素の削除（値を指定）
genes.remove("atpA")          # ["mnmG", "gyrB", "dnaN"]

# 要素の削除（インデックスを指定）
del genes[0]                   # ["gyrB", "dnaN"]
```

### 便利な操作

```python
numbers = [3, 1, 4, 1, 5, 9, 2, 6]

# ソート（元のリストは変えない）
sorted_nums = sorted(numbers)       # [1, 1, 2, 3, 4, 5, 6, 9]

# ソート（元のリスト自体を並び替え）
numbers.sort()

# 最大・最小・合計
print(max(numbers))    # 9
print(min(numbers))    # 1
print(sum(numbers))    # 31

# リストの連結
list_a = [1, 2, 3]
list_b = [4, 5, 6]
combined = list_a + list_b    # [1, 2, 3, 4, 5, 6]
```

---

## 2. タプル

**タプル**（`tuple`）は、リストに似ていますが**変更ができない**データ型です。`()` で作成します。

```python
# タプル（変更不可）
codon = ("ATG", "start")
print(codon[0])    # "ATG"
print(codon[1])    # "start"

# 以下はエラーになる
# codon[0] = "TAA"  # TypeError: 'tuple' object does not support item assignment
```

変更されたくないデータ（座標のペア、辞書のキーなど）にはタプルを使います。また、複数の値を一度に返す関数の戻り値としても頻繁に使われます。

```python
# 複数の値をまとめて返す関数の例
def get_gc(seq):
    g = seq.count("G")
    c = seq.count("C")
    return g, c   # タプルとして返している

g_count, c_count = get_gc("ATGCGATCGA")
print(g_count, c_count)    # 3 2
```

---

## 3. 辞書

### 辞書とは

**辞書**（`dict`）は、**キー**と**値**のペアでデータを管理するデータ型です。`{}` で作成します。

```python
# コドン表の一部
codon_table = {
    "ATG": "Met",
    "TAA": "Stop",
    "TAG": "Stop",
    "TGA": "Stop",
    "GCT": "Ala",
}

# キーで値を取得
print(codon_table["ATG"])    # "Met"
print(codon_table["GCT"])    # "Ala"
```

### 基本操作

```python
gene_info = {
    "name": "mnmG",
    "product": "tRNA modification enzyme",
    "length_bp": 1881,
}

# 値の取得（キーが存在しない場合にデフォルト値を返す）
print(gene_info.get("function", "unknown"))  # "unknown"

# 要素の追加・変更
gene_info["function"] = "tRNA modification"

# 要素の削除
del gene_info["length_bp"]

# キーの存在確認
print("name" in gene_info)          # True
print("length_bp" in gene_info)     # False
```

### キー・値の一覧

```python
gene_info = {"name": "mnmG", "product": "tRNA modification enzyme", "length_bp": 1881}

print(list(gene_info.keys()))      # ['name', 'product', 'length_bp']
print(list(gene_info.values()))    # ['mnmG', 'tRNA modification enzyme', 1881]
print(list(gene_info.items()))     # [('name', 'mnmG'), ('product', 'tRNA modification enzyme'), ('length_bp', 1881)]
```

`.items()` はキーと値のペア（タプル）のリストを返します。

---

## 4. 集合

### 集合とは

**集合**（`set`）は、**重複のない**要素の集まりです。順序は保持されません。

```python
# 作成（重複は自動的に除去される）
genes = {"mnmG", "gyrB", "dnaN", "mnmG"}
print(genes)    # {'mnmG', 'gyrB', 'dnaN'}（重複が除かれる）

# 空の集合は set() で作る（{} は空の辞書になってしまう）
empty_set = set()
```

### 基本操作

```python
genes = {"mnmG", "gyrB", "dnaN"}

print(len(genes))           # 3
print("gyrB" in genes)      # True（存在確認はリストより高速）

# 追加・削除
genes.add("atpA")
genes.discard("dnaN")       # 存在しなくてもエラーにならない
```

### 集合演算

```python
a = {"mnmG", "gyrB", "dnaN"}
b = {"gyrB", "atpA", "dnaN"}

print(a | b)   # 和集合（どちらかに含まれる）  → {'mnmG', 'gyrB', 'dnaN', 'atpA'}
print(a & b)   # 積集合（両方に含まれる）      → {'gyrB', 'dnaN'}
print(a - b)   # 差集合（aにあってbにない）    → {'mnmG'}
```

---

## まとめ

| データ型 | 表記 | 順序 | 変更 | 重複 | 主な用途 |
|---------|------|------|------|------|---------|
| `list` | `[...]` | あり | 可 | 可 | 順序のある要素の集まり |
| `tuple` | `(...)` | あり | 不可 | 可 | 変更したくない固定データ |
| `dict` | `{key: val}` | あり | 可 | キーは不可 | キーで値を検索 |
| `set` | `{...}` | なし | 可 | 不可 | ユニークな要素・集合演算 |
