# NK-HanDic: morphological analysis dictionary for North Korean language

[한국어 Readme](README.md)

NK-HanDic(북한딕)は，形態素解析エンジン「[MeCab](https://taku910.github.io/mecab/)」で朝鮮民主主義人民共和国の言語（ここでは「朝鮮語」と呼ぶことにします）を解析するための辞書です．
21万を超える項目で構成されており，「로동신문」（労働新聞）や北朝鮮の朝鮮語教材など，書きことばを中心としたデータで学習，構築されています．

著作権の問題から学習データは公開しませんが，学習モデルをパッケージに同梱しています．

## Requirements

  - MeCab
  - Python or Perl

## インストール

git clone

```console
$ git clone https://github.com/okikirmui/nkhandic.git
```

あるいはZIPファイルをダウンロード

seedディレクトリに移動

```console
$ cd nkhandic/seed/
```

ZIPファイルの場合，解凍してseedディレクトリに移動

```console
$ cd nkhandic-master/seed/
```

バイナリ辞書の作成

```console
$ /usr/local/libexec/mecab/mecab-dict-index -f utf8 -t utf8
```

パラメータ学習用のモデルファイル`model`を使って，配布用辞書を作成（`/usr/local/lib/mecab/dic/nkhandic`にインストールする場合）

```console
$ /usr/local/libexec/mecab/mecab-dict-gen -o /usr/local/lib/mecab/dic/nkhandic -m model
```

解析用バイナリ辞書の作成

```console
$ cd /usr/local/lib/mecab/dic/nkhandic
$ /usr/local/libexec/mecab/mecab-dict-index -f utf8 -t utf8
```

解析する時，実際に必要なファイルは`char.bin`，`dicrc`，`matrix.bin`，`sys.dic`，`unk.dic`です．

## 使い方

### 実行時に辞書を指定

MeCabの実行時に`-d`オプションで辞書ファイルへのパスを指定することができます．

```console
$ mecab -d /usr/local/lib/mecab/dic/nkhandic
```

この方法だと，実行するたびに辞書を指定する必要があります．

### 設定ファイルで辞書を指定

ホームディレクトリに`.mecabrc`というファイルを作成し，`dicdir`にNK-HanDic辞書ファイルへのパスを記述して，辞書を指定することができます．

```text
dicdir = /usr/local/lib/mecab/dic/nkhandic
```

この方法では，解析の際，常にNK-HanDicが使われることになります．

### 入力

NK-HanDicはUTF-8エンコーディングされたテキストを入力として受け取り，形態素解析を行います．
入力としては，完成形ハングル（Hangul Syllables：ハングル音節文字領域の文字）ではなく，初声・中声・終声に分けた組み合わせ形（ハングル字母領域の文字）で記述する必要があります．
例えば完成形ハングルの'몸'（U+BAB8）は，ハングル字母領域の文字を使って'ㅁ'（U+1106），'ㅗ'（U+1169），'ㅁ'（U+11B7）に分けて，入力として与えます．

こうした変換処理は，任意のスクリプトを使って処理して構いません．
このパッケージでは，Perlスクリプト`k2jamo.pl`とPythonスクリプト`k2jamo.py`を提供しています．`tools`ディレクトリを参照してください．

`k2jamo.pl`で`input.txt`を分析する場合：

```console
$ perl k2jamo.pl input.txt | mecab -d /usr/local/lib/mecab/dic/nkhandic
```

あるいは文を直接入力する場合：

```console
$ echo "겨울 방학 때 뭐 했어요?" | perl k2jamo.pl | mecab -d /usr/local/lib/mecab/dic/nkhandic
```

### トークン化(tokenize)

出力フォーマットを指定する`-O`オプションを用いることで，トークン化処理を行うことが出来ます．
出力フォーマットとして，`tokenize`を指定します．

```console
$ echo "모든 치료조건과 환경이 그쯘하게 갖추어진 료양소의 구내에 야외휴식터를 번듯하게 꾸리고 수종이 좋은 나무들과 꽃관목들로 이채로운 원림경관을 조성하여 주변풍치를 한껏 돋구었다." | perl k2jamo.pl | mecab -d /usr/local/lib/mecab/dic/nkhandic -O tokenize
모든 치료 조건 과 환경 이 그쯘하 게 갖추어 지 ㄴ 료양소 의 구내 에 야외 휴식터 를 번듯 하 게 꾸리 고 수종 이 좋으 ㄴ 나무 들 과 꽃 관목 들 로 이채 로우 ㄴ 원림 경관 을 조성 하여 주변 풍치 를 한껏 돋구어 ㅆ 다 .
```

## 使い方(Python)

[PyPI](https://pypi.org/project/nkhandic/)にて`nkhandic`パッケージを公開しました．
`mecab-python3`パッケージと，入力を変換するための`jamotools`といったパッケージと共に使います．

インストール:

```bash
pip install nkhandic mecab-python3 jamotools
```

使用例:

```Python
import MeCab
import nkhandic
import jamotools

mecaboption = f'-r /dev/null -d {nkhandic.DICDIR}'

tokenizer = MeCab.Tagger(mecaboption)
tokenizer.parse('')

# 労働新聞 2024年5月1日付け社説
sentence = u'경애하는 총비서동지에 대한 절대적인 충성심을 지니고 당중앙의 구상과 결심을 철저한 실천행동으로 받들어나가야 한다.'

jamo = jamotools.split_syllables(sentence, jamo_type="JAMO")

node = tokenizer.parseToNode(jamo)
while node:
    print(node.surface, node.feature)
    node = node.next
```

## 品詞に関する情報

品詞に関する情報については，[品詞情報](docs/pos_detail.md)文書を参照してください．

## 辞書の構築と辞書項目

### 辞書の構築方法

辞書構築においてはMeCabの「再学習」機能（[オリジナル辞書/コーパスからのパラメータ推定](https://taku910.github.io/mecab/learn.html)を参照）を用いています．
現代韓国語解析用辞書の「[HanDic](https://github.com/okikirmui/handic)」辞書データに，以下で説明する朝鮮語項目を`z_NK.csv`ファイルにまとめて追加し，HanDic学習モデルを元にして朝鮮語データで再学習しました．
具体的には以下のような作業を行っています．

```console
# mecab-dict-indexを行った後
# handic_model：HanDic学習用モデル，corpus.txt：朝鮮語学習用データ
# 全てのファイルが同じディレクトリにあるという場合
# 再学習実行
$ /usr/local/libexec/mecab/mecab-cost-train -p 2 -M handic_model -c 1.0 corpus.txt model
# 配布用辞書の作成（finalディレクトリに出力する場合）
$ /usr/local/libexec/mecab/mecab-dict-gen -o final -d . -m model
# 解析用バイナリ辞書作成
$ cd final
$ /usr/local/libexec/mecab/mecab-dict-index -f utf8 -t utf8
```

### 学習用データ

再学習に使用した朝鮮語データの現況は以下のとおりです．

| カテゴリ   |         | 記事・冊数 | 文数 |
|------------|---------|-------:|-----------:|
| 労働新聞     | 社会文化生活  |     24|      428|
|            | 前進する朝鮮   |     17|     538|
|            | 社説 |     18|     325|
|            | 日付別記事     |    215 |   3,534 |
| 朝鮮語教材  |         |    24|           11,782|
| 合計 |         |     |     16,607|

詳しいリストは[学習用データ](docs/data_list_ja.md)を参照のこと．

### 辞書に登録した項目

辞書項目は，現代韓国語分析用辞書[HanDic](https://github.com/okikirmui/handic)を元に，以下の資料を参照して項目を追加しました．

  - [우리말샘](https://opendict.korean.go.kr/): 2024年4月のデータで，「북한어」と表示されている項目
  - 산업연구원DB [KIET 북한 산업·기업 DB](http://nkindustry.kiet.re.kr/comp/list.do): 北朝鮮の企業目録
  - [통일부 북한정보포털](https://nkinfo.unikorea.go.kr/nkp/word/nkword.do): 「북한 지도」で「백화점」（デパート），‘상점’（商店），‘휴양소’（休養所），‘호텔’（ホテル），‘려관’（旅館），‘야영소’（野営所），‘대학’（大学）などの検索語でヒットした項目
  - 上記「북한정보포털」から取得した『북한 기관별 인명록(본권)』，『북한 기관별 인명록(별책)』（北朝鮮機関別人名録（本巻，別冊））2022，2023に記載された機関名，人名の一部
  - [HanDic](https://github.com/okikirmui/handic)データのうち，用言の一部を修正

『조선말대사전』（2017年）を参照して追加した項目もあります．

## Author

  - Yoshinori Sugai(스가이 요시노리/須賀井義教, Kindai University)

## Reference

  - 스가이 요시노리(2024), `북한 조선어 형태소 분석 사전 구축에 관한 연구'(朝鮮語形態素解析辞書の構築に関する研究), 한국사전학(韓国辞書学) 제44호, 서울: 한국사전학회, pp.33-63.

## Copyrights

Copyright (c) 2024- Yoshinori Sugai. All rights reserved.

''NK-HanDic'' is under [BSD-3-Clause](https://opensource.org/licenses/BSD-3-Clause).
