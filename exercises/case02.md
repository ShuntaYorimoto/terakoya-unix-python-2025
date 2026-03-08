# レポート課題2: 毒腺特異的発現遺伝子の解析

---

## 背景

寄生蜂 *Asobara japonica* は宿主ショウジョウバエ（*Drosophila* 属）に寄生するコマユバチの一種です。メスは宿主幼虫に産卵する際、**毒液（venom）** を注入します。この毒液は宿主の **翅原基（imaginal disc）** に影響し、アポトーシスや細胞周期の停止を引き起こします。この現象は **IDD（Imaginal Disc Degradation、翅原基縮退）** と呼ばれ、宿主の発生を操作することで寄生を成功させていると考えられています。

Kamiyama et al.（2025, *Science Advances*）は、毒液を産生する**毒腺 (venom gland)** と他組織の RNA-seq 比較解析（EdgeR による DEG 解析）および RNAi スクリーニングにより、IDD に必須の毒液タンパク質として **IDDF1**（`AJ1_g7424`）と **IDDF2**（`AJ1_g8061`）を同定しました。この2つのタンパク質はいずれも **DUF4803**（Domain of Unknown Function 4803）と呼ばれる機能未知のドメインをもちます。

本課題では、頼本が再解析したデータを用いて、**毒腺で特異的に高発現している遺伝子 (DEG: Differentially Expressed Genes) の抽出やアノテーション情報を統合した毒液タンパク質候補の探索**を体験してもらいます。

---

## データ

| ファイル | 内容 |
|---------|------|
| `vg_vs_cc_aj_edgeR.tsv` | EdgeR によるDEG解析の結果（タブ区切り）。1列目（行名）が gene_id、以降の列: `logFC`, `logCPM`, `F`, `PValue`, `FDR` |
| `AJ1.aa.interproscan.tsv` | InterProScan によるタンパク質ドメイン解析結果（タブ区切り・ヘッダーなし・15列） |
| `AJ1.cds.fa` | *A. japonica* の全 CDS 配列（FASTA 形式）。配列 ID の形式: `AJ1_g7424.t1`（遺伝子 ID + isoform 番号） |

### `vg_vs_cc_aj_edgeR.tsv` の注意

このファイルは R の `write.csv()` / `write.table()` で出力したもので、**1列目（gene_id）にはヘッダーがありません**。
pandas で読み込むと gene_id は自動的に行インデックスになります。

```python
df = pd.read_csv("vg_vs_cc_aj_edgeR.tsv", sep="\t", index_col=0)
df.index.name = "gene_id"
df = df.reset_index()
```

### `AJ1.aa.interproscan.tsv` のカラム一覧

| 列番号 | 内容 | 例 |
|--------|------|-----|
| 1 | Protein accession | `AJ1_g7424.t1` |
| 2 | Sequence MD5 digest | `14086411a2cdf1c4...` |
| 3 | Sequence length | `3418` |
| 4 | Analysis | `Pfam` / `Gene3D` / `PRINTS` など |
| 5 | Signature accession | `PF16061` |
| 6 | Signature description | `Domain of unknown function (DUF4803)` |
| 7 | Start location | `10` |
| 8 | Stop location | `200` |
| 9 | Score（e-value） | `3.1E-52` |
| 10 | Status | `T` |
| 11 | Date | `05-03-2026` |
| 12 | InterPro accession | `IPR031718` |
| 13 | InterPro description | `DUF4803` |
| 14 | GO annotations（任意列） | `GO:0005515(InterPro)` |
| 15 | Pathways annotations（任意列） | `REACT_71` |

---

## 準備

```bash
mkdir ~/work/case02
cp ~/data/case02/* ~/work/case02/
cd ~/work/case02
ls
```

作業ディレクトリに移動してから JupyterLab を開き、以下の課題を解いてください。

---

## 問1: 毒腺特異的 DEG の抽出

`vg_vs_cc_aj_edgeR.tsv` を pandas で読み込み、以下の条件を**両方**満たす遺伝子を抽出してください。

- **FDR < 0.05**（統計的に有意）
- **logFC >= 1**（毒腺での発現量が対照組織の 2 倍以上）

**回答する内容:**
1. 使用したコード
2. 条件を満たす遺伝子数（DEG 数）

---

## 問2: DUF4803 ドメインを持つ遺伝子の解析

`AJ1.aa.interproscan.tsv` を pandas で読み込み、以下の条件を**両方**満たす行を抽出してください。

- **4列目（Analysis）が `"Pfam"`** である
- **6列目（Signature description）が `"DUF4803"` を含む**

次に、1列目の `protein_id`（例: `AJ1_g7424.t1`）から **gene_id**（`AJ1_g7424`）を取り出し、
問1で使用した EdgeR の結果テーブルと照合してください。

> ヒント: 11章で学んだ `merge()` を使うと2つのテーブルを結合できます。

> **補足1 — 特定列のみ読み込む（`usecols`）**
> `pd.read_csv()` に `usecols=[0, 3, 5]` を渡すと、1・4・6列目だけを読み込めます（`usecols` には Python の慣例に従い **0始まり**の番号を指定します）。列数が行によって可変なファイルでも、位置指定なので影響を受けません。同時に `names=["protein_id", "analysis", "description"]` で列名を付けておくと後の操作が楽になります。
>
> **補足2 — DataFrame の文字列操作（`.str` アクセサ）**
> pandas の列（Series）に対して文字列メソッドを適用するには、`.str.メソッド名()` のように `.str` を経由します。たとえば `protein_id` 列の値を `.` で分割して先頭要素を取り出すには:
> ```python
> df["gene_id"] = df["protein_id"].str.split(".").str[0]
> ```
> `str.split(".")` が各値をリストに分割し、`.str[0]` でリストの先頭要素（遺伝子 ID 部分）を取り出します。
>
> **補足3 — `str.contains()` の `na=False`**
> `description` 列に空欄（NaN）が含まれる場合、`str.contains("DUF4803")` はエラーになります。`na=False` を指定すると NaN の行を `False` として扱い、エラーを回避できます:
> ```python
> df[df["description"].str.contains("DUF4803", na=False)]
> ```

**回答する内容:**
1. 使用したコード
2. *A. japonica* ゲノム中で DUF4803 ドメイン（Pfam）を持つ遺伝子の総数
3. そのうち問1の DEG に含まれる遺伝子数

---

## 問3: MA plot の作成

logCPM（x 軸）vs. logFC（y 軸）の MA plot を作成し、`ma_plot.png` として保存してください。

点の色は以下のルールで色分けしてください。

| 条件 | 色 |
|-----|----|
| DEG でない（FDR >= 0.05 または logFC < 1） | 灰色 |
| DEG（DUF4803 ドメインなし） | 赤 |
| DEG かつ DUF4803 ドメインを持つ | 青 |

さらに、以下の要素を追加してください。

- `plt.axhline(1, ...)` で logFC = 1 の閾値線（破線）を追加
- 遺伝子 `AJ1_g7424`（IDDF1）と `AJ1_g8061`（IDDF2）に `plt.text()` でラベルを付ける

> **補足 — 3色の色分け**
> ex12 の MA plot（問2）では 2色をリスト内包表記で実現しました。3色の場合も同じ考え方で、条件を入れ子にすれば実現できます。たとえば各遺伝子に色を割り当てる関数を定義して `df.apply()` で適用する方法があります:
> ```python
> def assign_color(row):
>     is_deg = (row["FDR"] < 0.05) and (row["logFC"] >= 1)
>     if is_deg and row["gene_id"] in duf_genes:
>         return "blue"
>     elif is_deg:
>         return "red"
>     return "gray"
>
> df["color"] = df.apply(assign_color, axis=1)
> ```
> `df.apply(関数, axis=1)` は各行（`axis=1`）に対して関数を適用し、戻り値を新しい列として格納します。色列ができたら、色ごとにグループを絞り込んで `plt.scatter()` を複数回呼ぶと、凡例（`label=`）を正しく付けられます。1回の呼び出しで全点を渡すと `label=` が1つしか登録できないため、以下のように色ごとに分けて呼ぶのがポイントです:
> ```python
> for color, label in [("gray", "non-DEG"), ("red", "DEG"), ("blue", "DEG + DUF4803")]:
>     subset = df[df["color"] == color]
>     plt.scatter(subset["logCPM"], subset["logFC"], c=color, label=label)
>
> plt.legend()  # 3色それぞれの凡例が作られる
> ```

**回答する内容:**
1. 使用したコード
2. 作成した MA plot の図

---

## 問4: DUF4803 DEG の上位5遺伝子

問2で得た「DUF4803 ドメインを持つ DEG」を **logFC の降順**にソートし、
上位5遺伝子の `gene_id` と `logFC` を表示してください。

**回答する内容:**
1. 使用したコード
2. 上位5遺伝子の一覧（gene_id と logFC）

---

## 問5: BioPython による CDS 配列の抽出

毒液タンパク質候補遺伝子の機能解析のために、CDS 配列が必要です。
問4で得た上位5遺伝子の CDS 配列を `AJ1.cds.fa` から抽出してください。

> **注意**: `AJ1.cds.fa` の配列 ID は `AJ1_g7424.t1` のように isoform 番号付きです。
> 各 gene_id に `.t1` を付加した ID（例: `AJ1_g7424.t1`）を検索対象としてください。

`SeqIO.parse()` で FASTA を読み込み、対象 ID に一致する配列を `top5_duf4803_cds.fasta` として保存してください。

**回答する内容:**
1. 使用したコード
2. 抽出した配列ファイル（`top5_duf4803_cds.fasta`）

---

## レポートに含めるべき内容

1. 各問の **使用したコード**
2. 各問の **数値的な結果**（DEG 数、DUF4803 遺伝子数など）
3. **MA plot の図**（問3）
4. 問4の **上位5遺伝子の一覧**
5. 抽出した配列ファイル（`top5_duf4803_cds.fasta`）（問5）

---
