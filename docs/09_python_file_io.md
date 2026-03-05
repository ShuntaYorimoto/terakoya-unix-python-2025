# 第9章 ファイル処理

---

## 目次

1. [ファイルの読み込み](#1-ファイルの読み込み)
2. [ファイルへの書き込み](#2-ファイルへの書き込み)
3. [実用例：バイオインフォマティクスデータのパース](#3-実用例バイオインフォマティクスデータのパース)

---

## 1. ファイルの読み込み

### open() と with 文

Python でファイルを読み書きするには `open()` 関数を使います。**`with` 文**を使うと、ファイルが自動的に閉じられるため安全です。

```python
with open("proteins.fasta", "r") as f:
    for line in f:
        print(line.strip())    # strip() で末尾の改行を除去
```

| モード | 意味 |
|--------|------|
| `"r"` | 読み込み（デフォルト） |
| `"w"` | 書き込み（上書き） |
| `"a"` | 追記 |

### 読み込み方法いろいろ

```python
# 全内容を1つの文字列として読む
with open("proteins.fasta", "r") as f:
    content = f.read()

# 全行をリストとして読む
with open("proteins.fasta", "r") as f:
    lines = f.readlines()
    print(len(lines))    # 行数

# 1行ずつ処理する（大きなファイルに向く）
with open("proteins.fasta", "r") as f:
    for line in f:
        line = line.strip()
        # 何らかの処理
```

> **ファイルサイズが大きい場合**は `for line in f:` の1行ずつ処理が安全です。`read()` や `readlines()` はファイル全体をメモリに読み込むため、数GB のゲノムファイルでは注意が必要です。

---

## 2. ファイルへの書き込み

```python
genes = ["mnmG", "gyrB", "dnaN"]

with open("gene_list.txt", "w") as f:
    for gene in genes:
        f.write(gene + "\n")    # 改行を忘れずに
```

> ⚠️ **注意**: `"w"` モードは既存のファイルを**上書き**します。追記したい場合は `"a"` モードを使いましょう。

---

## 3. 実用例：バイオインフォマティクスデータのパース

### FASTAファイルから配列名を抽出

```python
headers = []

with open("proteins.fasta", "r") as f:
    for line in f:
        if line.startswith(">"):
            headers.append(line.strip())

for header in headers:
    print(header)
```

### FASTAファイルを辞書として読み込む

```python
sequences = {}
current_id = None

with open("proteins.fasta", "r") as f:
    for line in f:
        line = line.strip()
        if line.startswith(">"):
            current_id = line[1:].split()[0]    # ">" を除き最初の単語をIDとする
            sequences[current_id] = ""
        else:
            sequences[current_id] += line

# 確認
for seq_id, seq in sequences.items():
    print(f"{seq_id}: {len(seq)} aa")
```

### BLAST タブ区切り出力のパース

BLAST の `-outfmt 7` 出力を Python で処理するパターンです。`strip()`・`split()`・型変換・`if` 文の総合的な活用例です。

```python
# eyeless_vs_human.fmt7.txt を読み込み、pident が 80% 以上のヒットを表示
with open("eyeless_vs_human.fmt7.txt", "r") as f:
    for line in f:
        if line.startswith("#"):
            continue    # コメント行はスキップ
        fields = line.strip().split("\t")    # タブで分割してリストに
        pident = float(fields[2])            # 3列目：一致率（%）
        evalue = float(fields[10])           # 11列目：E-value
        if pident >= 80.0:
            print(f"hit: {fields[1]}, pident: {pident}%, E-value: {evalue}")
```

| インデックス | 列名 | 内容 |
|------------|------|------|
| `fields[0]` | qseqid | クエリ配列名 |
| `fields[1]` | sseqid | ヒット配列名 |
| `fields[2]` | pident | 一致率（%） |
| `fields[10]` | evalue | E-value |

---

## まとめ

| 操作 | コード |
|------|--------|
| ファイルを1行ずつ読む | `for line in f:` |
| 末尾の改行を除去 | `line.strip()` |
| タブで分割 | `line.split("\t")` |
| ファイルに書き出す | `f.write(text + "\n")` |
