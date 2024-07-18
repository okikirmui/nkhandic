# NK-HanDic: morphological analysis dictionary for North Korean language

[한국어 Readme](README.md)

NK-HanDic(북한딕)は，形態素解析エンジン「[MeCab](https://taku910.github.io/mecab/)」で朝鮮民主主義人民共和国の言語（ここでは「朝鮮語」と呼ぶことにします）を解析するための辞書です．
20万を超える項目で構成されており，「로동신문」（労働新聞）や北朝鮮の朝鮮語教材など，書きことばを中心としたデータで学習，構築されています．

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

## 品詞に関する情報

品詞に関する情報については，[品詞情報](pos_detail.md)文書を参照してください．

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
/usr/local/libexec/mecab/mecab-cost-train -p 2 -M handic_model -c 1.0 corpus.txt model
# 配布用辞書の作成（finalディレクトリに出力する場合）
/usr/local/libexec/mecab/mecab-dict-gen -o final -d . -m model
# 解析用バイナリ辞書作成
cd final
/usr/local/libexec/mecab/mecab-dict-index -f utf8 -t utf8
```

再学習に使用した朝鮮語データは以下のとおりです（2,466文）．

#### 労働新聞（로동신문）

[ホームページ](http://www.rodong.rep.kp/ko/)の「사회문화생활」（社会文化生活）カテゴリに掲載された記事リストのうち，各日最初の記事を選択しました．
該当記事が長すぎる場合には，同じ日の次の記事を選択し，当日の記事がそれ一つしかない場合は，長くとも該当記事を選択しました．

  1. [고조되는 교육지원열의(2024/4/1, 5면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNC0wMS1OMDI1QDVAMUBAMEAxMw==)
  1. [전국식료품전시회-2024 폐막(2024/4/3, 4면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNC0wMy1OMDE0QDVAMUBAMEAxMA==)
  1. [자기 일터를 제손으로(2024/4/4), 5면](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNC0wNC1OMDI2QDVAMUBAMEA4==)
  1. [4월의 경축분위기를 더해줄 의의있는 계기 / 제33차 4월의 봄 친선예술축전 조직위원회 일군들과 나눈 이야기(2024/4/5, 2면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNC0wNS1OMDA1QDVAMUBAMEA1==)
  1. [귀항의 날에(2024/4/6, 6면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNC0wNi1OMDMyQDVAMUBAMEAz==)
  1. [일요일의 아침풍경(2024/4/7, 5면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNC0wNy1OMDIwQDVAMUBAMEAy==)
  1. [사회주의생활의 향기(2024/4/8, 6면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNC0wOC1OMDI5QDVAMUBAMEAx==)
  1. [제５９차 전국학생소년예술축전 개막(2024/4/9, 4면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNC0wOS1OMDE0QDVAMkBAMEAxOQ==)
  1. [전국방역, 보건부문학술토론회 진행(2024/4/11, 4면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNC0xMS1OMDE2QDVAMUBAMEAxMg==)
  1. [김책공업종합대학 종합실험교육관 개관식 진행(2024/4/15, 4면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNC0xNS1OMDE2QDVAMUBAMEA3==)
  1. [자강도에서 원탕료양소 개건(2024/4/17, 5면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNC0xNy1OMDA3QDVAM0BAMEAzNg==)
  1. [농장벌의 즐거운 휴식참(2024/4/18, 5면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNC0xOC1OMDI1QDVAM0BAMEAzMg==)
  1. [교육조건과 환경개선을 위한 사업 적극 추진(2024/4/19, 4면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNC0xOS1OMDE4QDVAMkBAMEAyNw==)
  1. [청년들의 무대 《애국과 청춘》 진행(2024/5/1, 5면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0wMS1OMDE4QDVAMkBAMEAyNg==) 첫 번째 기사가 너무 길어서 두 번째 기사를 채택
  1. [전국녀맹예술선동대경연 진행(2024/5/2, 4면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0wMi1OMDEwQDVAMkBAMEAxOQ==)
  1. [체육을 해도 정말 기백있게, 멋있게 한다(2024/5/3, 4면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0wMy1OMDE2QDVAMkBAMEAxNg==) 첫 번째, 두 번째 기사가 너무 길어서 세 번째 기사를 채택
  1. [２０２４년 봄철 국토환경보호부문 미학토론회 진행(2024/5/4, 5면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0wNC1OMDIxQDVAMUBAMEAxMA==) 첫 번째 기사가 너무 길어서 두 번째 기사를 채택
  1. [《문명의 별천지를 더욱 아름답게 가꾸어가렵니다》(2024/5/5, 4면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0wNS1OMDIxQDVAMUBAMEA4==) 너무 길지만 이날 기사가 이것밖에 없기 때문에 채택
  1. [사회주의시책속에 복된 삶을 누려가는 백살장수자들(2024/5/6, 4면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0wNi1OMDE5QDVAMUBAMEA2==)
  1. [우수한 교수자원이 농촌학교들에 보급된다(2024/5/7, 4면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0wNy1OMDE5QDVAMUBAMEAx==)
  1. [신심과 열정이 넘치는 우리 생활(2024/5/8, 6면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0wOC1OMDMyQDVAM0BAMEAzMg==)
  1. [제１６차 건축미학토론회 진행(2024/5/9, 4면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0wOS1OMDI2QDVAMkBAMEAyOA==)
  1. [한없이 소중한 내 고향, 내 조국을 위해(2024/5/11, 4면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0xMS1OMDE2QDVAMkBAMEAyMw==)
  1. [사회주의농촌에 펼쳐진 새 문명(2024/5/12, 5면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0xMi1OMDI3QDVAMkBAMEAyMg==)

#### 朝鮮語教材

各教材の対話文や講読部分を使用しました．

  - 「조선어기초(류학생용)」（朝鮮語基礎・留学生用，2017，金亨稷師範大学）
  - 「조선어(류학생용)1～5（朝鮮語・留学生用，2022，金日成総合大学）

### 辞書に登録した項目

辞書項目は，現代韓国語分析用辞書[HanDic](https://github.com/okikirmui/handic)を元に，以下の資料を参照して項目を追加しました．

  - [우리말샘](https://opendict.korean.go.kr/): 2024年4月のデータで，「북한어」と表示されている項目
  - 산업연구원DB [KIET 북한 산업·기업 DB](http://nkindustry.kiet.re.kr/comp/list.do): 北朝鮮の企業目録
  - [통일부 북한정보포털](https://nkinfo.unikorea.go.kr/nkp/word/nkword.do): 「북한 지도」で「백화점」（デパート），‘상점’（商店），‘휴양소’（休養所），‘호텔’（ホテル），‘려관’（旅館），‘야영소’（野営所），‘대학’（大学）などの検索語でヒットした項目
  - 上記「북한정보포털」から取得した『북한 기관별 인명록(별책)』（北朝鮮機関別人名録（別冊））2022に記載された機関名の一部
  - [HanDic](https://github.com/okikirmui/handic)データのうち，用言の一部を修正

## Author

  - Yoshinori Sugai(스가이 요시노리/須賀井義教, Kindai University)

## Copyrights

Copyright (c) 2024- Yoshinori Sugai. All rights reserved.

''NK-HanDic'' is under [BSD-3-Clause](https://opensource.org/licenses/BSD-3-Clause).
