# 第２章 バイオインフォマティクスのデータフォーマット

---

## 目次

1. [FASTA](#1-fasta)
2. [FASTQ](#2-fastq)
3. [GFF3 / GTF](#3-gff3--gtf)

---

## 1. FASTA

FASTA形式は、塩基配列やアミノ酸配列を記述するための最も基本的なフォーマットです。1エントリ（1配列）は次の2要素で構成されます。

```
>配列名 説明（タンパク質名、生物種名など）
ATGTCCAAACTAATTAATTTTGATGTAATAGTAGTAGGAGGGGGGCATGCAGGCACAGAAGCTTCTATTATTTCAGCAAA...
```

- **ヘッダ行**: `>` で始まる行。配列名が書かれる。配列名のあとにスペースを挟み、その配列の説明が書かれていることがある。
- **配列行**: ヘッダ行に続く1行以上の配列。1行に収まらない場合は複数行に分割される。

複数の配列を1つのファイルにまとめることができます（マルチFASTA）。

```
>BGI51261.1 MAG: chaperonin GroEL [Buchnera aphidicola (Ceratovacuna japonica)]
MAAKDVKFGNEARIKMLNGVNILADAVRVTLGPKGRNVVLDKSFGAPSITKDGVSVAREIELEDKFENMGAQMVKEVASK
ANDAAGDGTTTATLLAQAIVNEGLKAVASGMNPMDLKRGIDKAVINAVNELRKLSVPCSNSKAITQVGTISANADEKVGS
...
>BGI51348.1 MAG: molecular chaperone DnaK [Buchnera aphidicola (Ceratovacuna japonica)]
MSKIIGIDLGTTNSCVAIIDGNKTKVLENSEGDRTTPSTIAYTQEDEVLVGQPAKRQAITNPKNTLFAIKRLIGRKFTDK
EVQKDIKIMPYEIINSENGDAWIKIKNKKIAPPQISAEILKKMKKTAEEYTGEKINEAVITVPAYFNDAQRQATKDAGKI
...
```

**よく使われる拡張子**: `.fa`, `.fasta`, `.fna`（核酸配列）, `.faa`（アミノ酸配列）

---

## 2. FASTQ

FASTQ形式は、次世代シーケンサー（NGS）が出力するリード（短い配列断片）を格納するためのフォーマットです。FASTAと異なり、各塩基の**クオリティスコア**（塩基読み取りの信頼度）が付加されています。1リードは必ず **4行**で構成されます。

```
@M01037:80:000000000-G245Y:1:1101:15671:1867 1:N:0:GTAGAGGA+TTATGCGA
CCTACGGGTGGCAGCAGTGGGGAATATTGCACAATGGGCGAAAGCC      ← 塩基配列
+                                                   ← 区切り記号（固定）
>AA1>111ADDF10A0FGBGGGFGGHHHHHHHGGHHHHHGGEGGHH      ← クオリティスコア（ASCII文字）
```

| 行 | 内容 |
|----|------|
| 1行目 | `@` で始まるリードID |
| 2行目 | 塩基配列 |
| 3行目 | `+` のみ（区切り記号） |
| 4行目 | クオリティスコア（塩基配列と同じ長さ） |

**よく使われる拡張子**: `.fq`, `.fastq`

### クオリティスコア（Phredスコア）とは

クオリティスコア（Q）は、各塩基のベースコールが誤っている確率（*e*）を次の式で表したものです。

$$Q = -10 \log_{10}(e)$$

つまり、**Qスコアが高いほど塩基の読み取りが正確**であることを示します。

| Phredスコア | エラー確率 | 正確さ |
|------------|-----------|--------|
| Q10 | 1/10（10%）| 90% |
| Q20 | 1/100（1%）| 99% |
| Q30 | 1/1000（0.1%）| 99.9% |
| Q40 | 1/10000（0.01%）| 99.99% |

一般的にQ30以上が高品質とされており、多くの解析パイプラインでQ20〜Q30を閾値としてトリミングを行います。

**低クオリティリードを除去せずに解析を進めた場合の影響**：
- アライメントの精度が低下し、リードの大部分が使用できなくなる可能性がある
- 偽陽性のバリアントコール（変異の誤検出）が増加する
- 不正確な結論につながるリスクがある

### ASCII文字によるスコアの表現

**ASCII**（アスキー）とは、文字と数値の対応を定めた文字コード規格です。ASCIIコード0〜32はコンピュータ制御用の特殊文字（改行や制御信号など）に割り当てられており、テキストファイル中に直接書くことができません。そのため、Phredスコアをテキストとして表現するにはコード33以降の文字を使う必要があり、33を引くことでスコアに変換します。

$$\text{Phredスコア} = \text{ASCIIコード} - 33$$

| ASCII文字 | ASCIIコード | Phredスコア | エラー確率 |
|-----------|-----------|------------|-----------|
| `!` | 33 | 0 | 1（100%） |
| `+` | 43 | 10 | 10% |
| `5` | 53 | 20 | 1% |
| `?` | 63 | 30 | 0.1% |
| `I` | 73 | 40 | 0.01% |

---

## 3. GFF3 / GTF

**GFF3**（Generic Feature Format version 3）と **GTF**（Gene Transfer Format）は、ゲノム上の遺伝子・エクソン・CDS などの**アノテーション情報**を記述するためのフォーマットです。どちらもタブ区切りのテキストファイルで、構造はよく似ています。

**よく使われる拡張子**: `.gff`, `.gff3`, `.gtf`

### フォーマット（GFF3）

`#` で始まる行はコメント行です。データ行は **9列のタブ区切り**で構成されます。

```
##gff-version 3
#!gff-spec-version 1.21
#!processor NCBI annotwriter
#!genome-build Buchnera aphidicola (Ceratovacuna japonica) strain CjNOSY1 v. 1.0
#!genome-build-accession NCBI_Assembly:GCA_024349705.1
##sequence-region AP026065.1 1 414725
##species https://www.ncbi.nlm.nih.gov/Taxonomy/Browser/wwwtax.cgi?id=2950563
AP026065.1	DDBJ	region	1	414725	.	+	.	ID=AP026065.1:1..414725;Dbxref=taxon:2950563;Is_circular=true;collection-date=2017-09-03;country=Japan:Nagano%2C Norikura;environmental-sample=true;gbkey=Src;isolate=CjNOSY1;isolation-source=whole body of the eusocial aphid%2C Ceratovacuna japonica strain NOSY1;metagenome-source=insect metagenome;mol_type=genomic DNA;nat-host=Ceratovacuna japonica
AP026065.1	DDBJ	gene	1	1881	.	+	.	ID=gene-BucCj_0010;Name=mnmG;gbkey=Gene;gene=mnmG;gene_biotype=protein_coding;locus_tag=BucCj_0010
AP026065.1	DDBJ	CDS	1	1881	.	+	0	ID=cds-BGI51245.1;Parent=gene-BucCj_0010;Dbxref=NCBI_GP:BGI51245.1;Name=BGI51245.1;gbkey=CDS;gene=mnmG;inference=similar to AA sequence:RefSeq:WP_010895896.1;locus_tag=BucCj_0010;product=tRNA uridine-5-carboxymethylaminomethyl(34) synthesis enzyme MnmG;protein_id=BGI51245.1;transl_table=11
AP026065.1	DDBJ	gene	1994	2809	.	+	.	ID=gene-BucCj_0020;Name=atpB;gbkey=Gene;gene=atpB;gene_biotype=protein_coding;locus_tag=BucCj_0020
AP026065.1	DDBJ	CDS	1994	2809	.	+	0	ID=cds-BGI51246.1;Parent=gene-BucCj_0020;Dbxref=NCBI_GP:BGI51246.1;Name=BGI51246.1;gbkey=CDS;gene=atpB;inference=similar to AA sequence:RefSeq:WP_010895897.1;locus_tag=BucCj_0020;product=F0F1 ATP synthase subunit A;protein_id=BGI51246.1;transl_table=11
...
```

| 列 | フィールド名 | 内容 |
|----|------------|------|
| 1 | seqid | 染色体名・配列名 |
| 2 | source | アノテーションの出所（Ensembl, RefSeq など） |
| 3 | type | 特徴の種類（gene, mRNA, exon, CDS など） |
| 4 | start | 開始位置（**1始まり**） |
| 5 | end | 終了位置 |
| 6 | score | スコア（不明の場合は `.`） |
| 7 | strand | 鎖の方向（`+` または `-`） |
| 8 | phase | 読み枠のオフセット（CDS以外は `.`） |
| 9 | attributes | 付加情報（ID, Name, Parent など） |

### フォーマット（GTF）
```
#gtf-version 2.2
#!genome-build Buchnera aphidicola (Ceratovacuna japonica) strain CjNOSY1 v. 1.0
#!genome-build-accession NCBI_Assembly:GCA_024349705.1
AP026065.1	DDBJ	gene	1	1881	.	+	.	gene_id "BucCj_0010"; transcript_id ""; gbkey "Gene"; gene "mnmG"; gene_biotype "protein_coding"; locus_tag "BucCj_0010"; 
AP026065.1	DDBJ	CDS	1	1878	.	+	0	gene_id "BucCj_0010"; transcript_id "unassigned_transcript_1"; db_xref "NCBI_GP:BGI51245.1"; gbkey "CDS"; gene "mnmG"; inference "COORDINATES: ab initio prediction:MetaGeneAnnotator"; inference "similar to AA sequence:RefSeq:WP_010895896.1"; locus_tag "BucCj_0010"; product "tRNA uridine-5-carboxymethylaminomethyl(34) synthesis enzyme MnmG"; protein_id "BGI51245.1"; transl_table "11"; exon_number "1"; 
AP026065.1	DDBJ	start_codon	1	3	.	+	0	gene_id "BucCj_0010"; transcript_id "unassigned_transcript_1"; db_xref "NCBI_GP:BGI51245.1"; gbkey "CDS"; gene "mnmG"; inference "COORDINATES: ab initio prediction:MetaGeneAnnotator"; inference "similar to AA sequence:RefSeq:WP_010895896.1"; locus_tag "BucCj_0010"; product "tRNA uridine-5-carboxymethylaminomethyl(34) synthesis enzyme MnmG"; protein_id "BGI51245.1"; transl_table "11"; exon_number "1"; 
AP026065.1	DDBJ	stop_codon	1879	1881	.	+	0	gene_id "BucCj_0010"; transcript_id "unassigned_transcript_1"; db_xref "NCBI_GP:BGI51245.1"; gbkey "CDS"; gene "mnmG"; inference "COORDINATES: ab initio prediction:MetaGeneAnnotator"; inference "similar to AA sequence:RefSeq:WP_010895896.1"; locus_tag "BucCj_0010"; product "tRNA uridine-5-carboxymethylaminomethyl(34) synthesis enzyme MnmG"; protein_id "BGI51245.1"; transl_table "11"; exon_number "1"; 
AP026065.1	DDBJ	gene	1994	2809	.	+	.	gene_id "BucCj_0020"; transcript_id ""; gbkey "Gene"; gene "atpB"; gene_biotype "protein_coding"; locus_tag "BucCj_0020";
```

### GFF3 と GTF の違い

両者の主な違いは **9列目（attributes）の書き方**です。

**GFF3**:
```
ID=cds-BGI51245.1;Parent=gene-BucCj_0010;Dbxref=NCBI_GP:BGI51245.1;Name=BGI51245.1;gbkey=CDS;gene=mnmG;inference=similar to AA sequence:RefSeq:WP_010895896.1;locus_tag=BucCj_0010;product=tRNA uridine-5-carboxymethylaminomethyl(34) synthesis enzyme MnmG;protein_id=BGI51245.1;transl_table=11
```

**GTF**:
```
gene_id "BucCj_0010"; transcript_id "unassigned_transcript_1"; db_xref "NCBI_GP:BGI51245.1"; gbkey "CDS"; gene "mnmG"; inference "COORDINATES: ab initio prediction:MetaGeneAnnotator"; inference "similar to AA sequence:RefSeq:WP_010895896.1"; locus_tag "BucCj_0010"; product "tRNA uridine-5-carboxymethylaminomethyl(34) synthesis enzyme MnmG"; protein_id "BGI51245.1"; transl_table "11"; exon_number "1"; 
```

GFF3はキーと値を"="で区切り、セミコロンで終わる形式です。
GTFはキーと値を半角スペースと引用符で区切り、セミコロンで終わる形式です。

---
