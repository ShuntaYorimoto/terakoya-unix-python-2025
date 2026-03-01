# レポート課題1: エゴノネコアシアブラムシ共生細菌ゲノムの同定

---

## 背景

エゴノネコアシアブラムシ（*Ceratovacuna nekoashi*）は、**2種の共生細菌**を保有することが最近発見されました（Yorimoto et al., 2025, *bioRxiv*）。

- *Buchnera aphidicola*
- *Arsenophonus* sp.

多くのアブラムシの *Buchnera* ゲノムは、**1本の染色体と2本のプラスミド**からなります。

| *Buchnera* ゲノム構成 | 主な特徴 |
|----------------------|---------|
| Chromosome | 染色体。さまざまな代謝遺伝子などを含む |
| pTrp | プラスミド。トリプトファン合成に関わる *trpG* などを含む |
| pLeu | プラスミド。ロイシン合成に関わる *leuA* などを含む |

*C. nekoashi* の *Arsenophonus* ゲノムは、**chromosome と plasmid の2本**からなることがわかっています。

今回用意した `data/case01/symbionts_cn.fasta` は、*C. nekoashi* から得られた共生細菌ゲノムの配列を **contig1〜5 にリネームして1ファイルに統合したもの**です。どの contig がどのゲノム要素・どの共生菌に由来するかは伏せてあります。

参照として、近縁種 *C. japonica* の *Buchnera* ゲノムデータが `data/ex01/` に用意されています。

---

## データ

| ファイル | 内容 |
|---------|------|
| `data/case01/symbionts_cn.fasta` | *C. nekoashi* 共生細菌ゲノム |
| `data/ex01/buchnera_cj.fna` | *C. japonica* *Buchnera* ゲノム配列 |
| `data/ex01/buchnera_cj.faa` | *C. japonica* *Buchnera* タンパク質配列 |
| `data/ex01/buchnera_cj.gff` | *C. japonica* *Buchnera* アノテーション |

---

## 課題

以下の3問に答えてください。**ツールの選択と組み合わせは自分で考えること。**
コマンドや実行結果・考察をまとめてレポートとして提出してください。

---

### 問1：contig の同定

**contig1〜5 のうち、どれが以下に相当するかを同定し、判断の根拠を示してください。**

- *Buchnera* chromosome
- *Buchnera* pTrp プラスミド
- *Buchnera* pLeu プラスミド
- *Arsenophonus* ゲノム（2本。chromosome か plasmid かは区別しなくてよい）

以下の手順を参考にしてください。

1. `data/ex01/buchnera_cj.fna` の配列名行（`>` から始まる行）を確認し、どの配列が chromosome・pTrp・pLeu に対応するかを調べてください
2. `data/ex01/buchnera_cj.fna` を query、`data/case01/symbionts_cn.fasta` を db として BLAST 検索を実行してください
3. 検索結果をもとに、contig1〜5 のうち *Buchnera* の chromosome・pTrp・pLeu に相当するものを同定してください
4. 残る2つの contig を *Arsenophonus* ゲノム由来とみなしてください。これらの contig に対して `data/ex01/buchnera_cj.fna` の配列がヒットする場合、なぜそのようなヒットが生じるか考察してください

---

### 問2：*trpG* を用いた pTrp の確認

以下の手順で *trpG* 配列を取り出し、*C. nekoashi* 共生細菌ゲノムへの検索に使ってください。

1. `data/ex01/buchnera_cj.gff` を検索して、*trpG* の `protein_id` を調べてください
2. その `protein_id` を使って `data/ex01/buchnera_cj.faa` から *trpG* のアミノ酸配列を取り出してください
3. 取り出した配列を使って `data/case01/symbionts_cn.fasta` を検索してください
4. 問1の結論（pTrp の同定）と結果が一致するか確認し、考察してください

---

### 問3：*leuA* を用いた pLeu の確認

問2 と同様の手順で *leuA* についても解析し、pLeu の同定を確認してください。

---

## レポートに含めるべき内容

1. 使用したコマンドとその出力（スクリーンショットまたはコピー貼り付け）
2. 各問の結論と、結論に至った根拠
3. 問2・問3 の BLAST 結果が問1の結論と一致したかの考察

---
