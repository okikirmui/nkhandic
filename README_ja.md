# NK-HanDic: morphological analysis dictionary for North Korean language

한국어 Readme：[README.md](README.md)

NK-HanDic(북한딕)は，形態素解析エンジン「[MeCab](https://taku910.github.io/mecab/)」で朝鮮民主主義人民共和国の言語（ここでは「朝鮮語」と呼ぶことにします）を解析するための辞書です．
22万を超える項目で構成されており，「로동신문」（労働新聞）や北朝鮮の小説など，書きことばを中心としたデータで学習，構築されています．

著作権の問題から学習データは公開しませんが，学習モデルをパッケージに同梱しています．

## Quick Start(Python / pip)

Pythonパッケージの**`nkhandic`**を使えば，**辞書の構築なしに形態素解析を始める**ことができます．

### パッケージのインストール

```bash
pip install nkhandic mecab-python3 jamotools
```

### 形態素解析の例

```python
import nkhandic

text = "우리의 경애하는 총비서동지께서는 어쩌면 그리도 위대하신가."

print(nkhandic.pos_tag(text))
print(nkhandic.tokenize_hangul(text, mode="surface"))
print(nkhandic.convert_text_to_hanja_hangul(text))
```

**出力**

```python
[('우리03', 'NP'), ('의10', 'JKG'), ('경애01', 'NNG'), ('하다02', 'XSV'), ('는03', 'ETM'), ('총비서', 'NNG'), ('동지006', 'NNG'), ('께서', 'JKS'), ('는01', 'JX'), ('어쩌면', 'MAG'), ('그리도', 'MAG'), ('위대02', 'XR'), ('하다02', 'XSA'), ('시', 'EP'), ('ㄴ가', 'EF'), ('.', 'SF')]
['우리', '의', '경애', '하', '는', '총비서', '동지', '께서', '는', '어쩌면', '그리도', '위대', '하', '시', 'ㄴ가', '.']
우리의 敬愛하는 總秘書同志께서는 어쩌면 그리도 偉大하신가.
```

### Pythonパッケージ（`nkhandic`）

- PyPI: https://pypi.org/project/nkhandic/
- MeCab用Pythonラッパー（`mecab-python3`），ハングル字母の処理パッケージ（`jamotools`）とともに使用

## Dictionary Build

Pythonを使わない場合，ローカルでMeCabを使って解析を行う場合，解析用のバイナリ辞書を構築する必要があります．

### Requirements

  - MeCab
  - Python or Perl

> ⚠️ **注意**
>
> NK-HanDicは**入力として完成型ハングルを使いません．**  
> MeCabで形態素解析を行う前に**必ず字母（Jamo）単位に分解して**，入力としなければなりません．
> 
> 字母への変換を行わない場合，スペース区切りの文字列全体が記号として扱われてしまいます．

### 辞書構築の方法（要約）

`mecab-dict-index`，`mecab-dict-gen`などの位置は，`mecab-config --libexecdir`の出力を参照してください．
以下では`/usr/local/libexec/mecab`にあると仮定しています．

```bash
# git clone
git clone https://github.com/okikirmui/nkhandic.git
cd handic
# バイナリ辞書の作成
/usr/local/libexec/mecab/mecab-dict-index -f utf8 -t utf8
# 学習済みのモデルファイル`model`を使って配布用辞書を作成
# `/usr/local/lib/mecab/dic/nkhandic`ディレクトリに出力
/usr/local/libexec/mecab/mecab-dict-gen -o /usr/local/lib/mecab/dic/nkhandic -m model
# 解析用バイナリ辞書の作成
$ cd /usr/local/lib/mecab/dic/nkhandic
$ /usr/local/libexec/mecab/mecab-dict-index -f utf8 -t utf8
```

解析する時，実際に必要なファイルは`char.bin`，`dicrc`，`matrix.bin`，`sys.dic`，`unk.dic`です．

## 使い方

### 実行時に辞書を指定

MeCabの実行時に`-d`オプションで辞書ファイルへのパスを指定することができます．

```bash
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

```bash
$ perl k2jamo.pl input.txt | mecab -d /usr/local/lib/mecab/dic/nkhandic
```

あるいは文を直接入力する場合：

```bash
$ echo "다음과 같이 말씀하시였다." | perl k2jamo.pl | mecab -d /usr/local/lib/mecab/dic/nkhandic
```

### トークン化(tokenize)

出力フォーマットを指定する`-O`オプションを用いることで，トークン化処理を行うことが出来ます．
出力フォーマットとして，`tokenize`を指定します．

```bash
$ echo "다음과 같이 말씀하시였다." | perl k2jamo.pl | mecab -d /usr/local/lib/mecab/dic/nkhandic -O tokenize
```

出力：

```Text
다음 과 같이 말씀 하 시여 ㅆ 다 .
```

## 品詞に関する情報

品詞に関する情報については，[品詞情報](docs/pos_detail.md)文書を参照してください．

## 辞書の学習と辞書項目

### 辞書の学習方法

辞書構築においてはMeCabの「再学習」機能（[オリジナル辞書/コーパスからのパラメータ推定](https://taku910.github.io/mecab/learn.html)を参照）を用いています．
現代韓国語解析用辞書の「[HanDic](https://github.com/okikirmui/handic)」辞書データに，以下で説明する朝鮮語項目を`z_NK.csv`ファイルにまとめて追加し，HanDic学習モデルを元にして朝鮮語データで再学習しました．
具体的には以下のような作業を行っています．

`mecab-dic-index`配布用辞書を作成したあと，再学習を実行します．
すべてのファイルが同じディレクトリにあるとします．

使用するデータは：

- `handic_model`：HanDicのモデルファイル
- `corpus.txt`：朝鮮語の再学習用コーパスデータ

```bash
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

| 資料   |  分野  | 記事・冊数 | 文数 |
|------------|---------|-------:|-----------:|
| 労働新聞 | 社説 | 18 | 325 |
|  | 日付別記事 | 293 | 5,059 |
|  | 社会文化生活 | 24 | 428 |
|  | 前進する挑戦 | 17 | 538 |
| 朝鮮文学 | 小説 | 8 | 400 |
| **合計** |   | **360** | **6,750** |

詳しいリストは[学習用データ](docs/data_list_ja.md)を参照のこと．

> **注意**
>
> 以前のバージョンでは，学習用データに朝鮮語教材なども多数含めていましたが，解析率などを比較した上で，現在は上記のような構成にしています．

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

  - 스가이 요시노리(2024), [`북한 조선어 형태소 분석 사전 구축에 관한 연구'](http://doi.org/10.33641/kolex.2024..44.33)(朝鮮語形態素解析辞書の構築に関する研究), 한국사전학(韓国辞書学) 제44호, 서울: 한국사전학회, pp.33-63.

## Copyrights

Copyright (c) 2024- Yoshinori Sugai. All rights reserved.

''NK-HanDic'' is under [BSD-3-Clause](https://opensource.org/licenses/BSD-3-Clause).
