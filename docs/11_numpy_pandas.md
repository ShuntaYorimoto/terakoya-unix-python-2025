# 第11章 NumPy・Pandas

---

## 目次

1. [NumPy](#1-numpy)
2. [Pandas](#2-pandas)

---

## 1. NumPy

NumPy（Numerical Python）は、数値計算のための基本ライブラリです。**配列 (ndarray)** を使った高速な数値演算が可能で、Pandas・Matplotlib などの基盤になっています。

```python
import numpy as np
```

### 配列の作成

```python
# リストから配列を作成
counts = np.array([10, 25, 8, 43, 17])

# 0で初期化した配列
zeros = np.zeros(5)                    # [0. 0. 0. 0. 0.]

# 等間隔の数列
positions = np.arange(0, 100, 10)     # [ 0, 10, 20, ..., 90]

# 2次元配列（行列）
matrix = np.array([[1, 2, 3],
                   [4, 5, 6]])
print(matrix.shape)    # (2, 3) ← 2行3列
```

### 基本的な統計

```python
coverage = np.array([32, 45, 28, 51, 39, 47, 33, 60, 22, 41])

print(np.mean(coverage))      # 平均: 39.8
print(np.std(coverage))       # 標準偏差: 10.85...
print(np.median(coverage))    # 中央値: 40.0
print(np.min(coverage))       # 最小値: 22
print(np.max(coverage))       # 最大値: 60
```

### Boolean indexing（条件によるフィルタリング）

NumPy 配列は、条件式をそのままインデックスとして使えます。

```python
coverage = np.array([32, 45, 28, 51, 39, 47, 33, 60, 22, 41])

# 40以上の要素だけ取り出す
high_cov = coverage[coverage >= 40]
print(high_cov)               # [45 51 47 60 41]

# 条件を満たす要素数
print(np.sum(coverage >= 40)) # 5

# 30未満の位置を調べる
low_positions = np.where(coverage < 30)
print(low_positions)          # (array([2, 8]),) ← インデックス2と8
```

### 2次元配列と axis

バイオインフォマティクスでは、**遺伝子 × サンプル**の行列形式でデータを扱うことが多いです。

```python
# 発現量データ（行: 遺伝子, 列: サンプル）
expression = np.array([
    [120, 135, 118],   # mnmG
    [  8,  12,   6],   # gyrB
    [450, 420, 480],   # atpA
    [ 55,  60,  52],   # dnaN
])

print(expression.shape)    # (4, 3) ← 4遺伝子 × 3サンプル

# 各遺伝子の平均発現量（行方向 = axis=1）
gene_means = np.mean(expression, axis=1)
print(gene_means)    # [124.3   8.7 450.   55.7]

# 各サンプルの平均発現量（列方向 = axis=0）
sample_means = np.mean(expression, axis=0)
print(sample_means)  # [158.25  156.75  164.  ]

# 平均発現量が100以上の遺伝子を抽出
high_expr = expression[gene_means >= 100]
print(high_expr.shape)    # (2, 3) ← mnmG, atpA が該当
```

---

## 2. Pandas

Pandas は、表形式（TSV/CSV）データを扱うためのライブラリです。スプレッドシートのような **DataFrame** を使って、データの読み込み・集計・フィルタリングが簡単に行えます。

```python
import pandas as pd
```

### TSV / CSV ファイルの読み込み

```python
# タブ区切りファイル（BLAST結果など）の読み込み
df = pd.read_csv("blast_results.tsv", sep="\t")

# ヘッダなしのファイル（列名を指定）
cols = ["qseqid", "sseqid", "pident", "length", "mismatch",
        "gapopen", "qstart", "qend", "sstart", "send", "evalue", "bitscore"]
df = pd.read_csv("eyeless_vs_human.fmt6.txt", sep="\t", header=None, names=cols)
```

### 基本操作

```python
print(df.shape)         # (行数, 列数)
print(df.head())        # 先頭5行を表示
print(df.tail(3))       # 末尾3行を表示
print(df.dtypes)        # 各列のデータ型
print(df.describe())    # 数値列の基本統計
```

### 列の参照と計算

```python
# 列を取り出す（Series として返る）
evalues = df["evalue"]

# 新しい列を追加
df["log_evalue"] = -np.log10(df["evalue"].replace(0, 1e-300))

# 複数列の参照
subset = df[["qseqid", "sseqid", "pident", "evalue"]]
```

### フィルタリング

```python
# Evalueが 1e-10 未満の行を抽出
significant = df[df["evalue"] < 1e-10]

# pident が 80% 以上かつ Evalueが 1e-5 未満
filtered = df[(df["pident"] >= 80) & (df["evalue"] < 1e-5)]

# sseqid に特定の文字列を含む行を抽出
pax_hits = df[df["sseqid"].str.contains("PAX")]
```

### ソートと集計

```python
# Evalueで昇順ソート
df_sorted = df.sort_values("evalue")

# pident で降順ソート
df_sorted = df.sort_values("pident", ascending=False)

# 上位 10 件を取得
top10 = df.sort_values("evalue").head(10)
```

### groupby による集計

```python
# クエリごとにヒット数を集計
hit_counts = df.groupby("qseqid")["sseqid"].count()
print(hit_counts)

# クエリごとの最良ヒット（最小E）
best_hits = df.sort_values("evalue").groupby("qseqid").first().reset_index()
```

### DataFrame のマージ

2つの DataFrame を共通のキー列で結合（JOIN）します。DEG（Differentially Expressed Genes：発現変動遺伝子）結果とアノテーション情報を組み合わせるときなどに使います。

```python
import pandas as pd

# DEG 結果（edgeR 出力を模した仮データ）
deg = pd.DataFrame({
    "gene_id":  ["mnmG", "gyrB", "dnaN", "atpA", "leuA", "trpG", "atpD", "atpG"],
    "logFC":    [ 2.3,   -1.8,   0.4,   -2.9,    3.1,    1.2,   -0.3,    0.8],
    "logCPM":   [ 8.1,    7.4,   6.9,    9.2,    5.8,    6.1,    7.7,    8.0],
    "PValue":   [0.0003, 0.008, 0.38,  0.0001,  0.002,  0.07,   0.55,   0.12],
    "FDR":      [0.002,  0.04,  0.62,   0.001,   0.01,  0.21,   0.78,   0.34],
})

# アノテーション情報（eggnog-mapper 出力を模した仮データ）
annot = pd.DataFrame({
    "gene_id":      ["mnmG", "gyrB", "dnaN", "atpA", "leuA", "trpG", "atpD", "atpG"],
    "Description":  ["tRNA modification enzyme MnmG",
                     "DNA topoisomerase subunit B",
                     "DNA polymerase III subunit beta",
                     "ATP synthase subunit alpha",
                     "2-isopropylmalate synthase",
                     "Anthranilate synthase component II",
                     "ATP synthase subunit beta",
                     "ATP synthase subunit gamma"],
    "PFAMs":        ["MnmG_MnmE,FAD_binding_3", "TOPRIM,DNA_topoisoIV",
                     "DNA_clamp", "ATP-synt_ab", "LeuA_dimer", "Anth_synt_II_N",
                     "ATP-synt_ab", "ATP-synt_ab_C"],
    "COG_category": ["J", "L", "L", "C", "H", "E", "C", "C"],
})

# LEFT JOIN でマージ（アノテーションがない遺伝子も残す）
merged = deg.merge(annot, on="gene_id", how="left")
print(merged.head())
```

| `how=` | 意味 |
|--------|------|
| `"left"` | 左のDF（deg）の全行を残す。右（annot）に対応がなければ NaN |
| `"inner"` | 両方に存在する行のみ残す |
| `"outer"` | 両方の全行を残す |

```python
# FDR < 0.05 の有意な遺伝子を logFC 降順で表示
sig = merged[merged["FDR"] < 0.05].sort_values("logFC", ascending=False)
print(sig[["gene_id", "logFC", "FDR", "Description"]])

# ATP合成酵素ドメインを持つ遺伝子だけ抽出
atp_genes = merged[merged["PFAMs"].str.contains("ATP-synt", na=False)]
print(atp_genes[["gene_id", "logFC", "PFAMs"]])
```

### ファイルへの書き出し

```python
# TSVとして保存
filtered.to_csv("filtered_blast.tsv", sep="\t", index=False)

# CSVとして保存
filtered.to_csv("filtered_blast.csv", index=False)
```

### 実用例：BLAST 結果の読み込みと集計

```python
import pandas as pd

# BLAST 結果を読み込む
cols = ["qseqid", "sseqid", "pident", "length", "mismatch",
        "gapopen", "qstart", "qend", "sstart", "send", "evalue", "bitscore"]
df = pd.read_csv("eyeless_vs_human.fmt6.txt", sep="\t", header=None, names=cols)

print(f"総ヒット数: {len(df)}")
print(f"ユニークなヒット遺伝子数: {df['sseqid'].nunique()}")

# Evalue 1e-10 未満に絞る
sig = df[df["evalue"] < 1e-10]
print(f"\nEvalue < 1e-10 のヒット数: {len(sig)}")

# pident 降順で上位5件を表示
print("\n上位5ヒット（pident 降順）:")
print(df.sort_values("pident", ascending=False)[["sseqid", "pident", "evalue"]].head())
```

---

## まとめ

| ライブラリ | 主な用途 |
|-----------|---------|
| NumPy | 数値配列・統計計算 |
| Pandas | 表形式データの操作 |
