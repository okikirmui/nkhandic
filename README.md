# NK-HanDic: morphological analysis dictionary for North Korean language

[日本語Readme](README_ja.md)

NK-HanDic(북한딕)은 형태소 분석 엔진 [MeCab](https://taku910.github.io/mecab/)(메카부)로 북한의 언어(조선어)를 분석하기 위해 개발된 분석 사전입니다.
20만 개가 넘는 항목으로 구성되어 있고, 로동신문이나 북한의 조선어 교재 등 문어를 중심으로 한 데이터로 학습, 구축되었습니다.

저작권 문제로 학습 데이터 자체는 공개하지 않지만 모델 파일은 패키지에 포함되어 있습니다.

## Requirements

  - MeCab
  - Python or Perl

## 설치

git clone

```console
$ git clone https://github.com/okikirmui/nkhandic.git
```

혹은 ZIP파일을 다운로드

seed 디렉토리로 이동

```console
$ cd nkhandic/seed/
```

ZIP 파일을 다운받은 경우, 압축을 풀고 seed 디렉토리로 이동

```console
$ cd nkhandic-master/seed/
```

Binary 사전 작성

```console
$ /usr/local/libexec/mecab/mecab-dict-index -f utf8 -t utf8
```

파라미터 학습용 모델 파일 `model`을 사용하여 배포용 사전을 작성(`/usr/local/lib/mecab/dic/nkhandic`에 설치할 경우)

```console
$ /usr/local/libexec/mecab/mecab-dict-gen -o /usr/local/lib/mecab/dic/nkhandic -m model
```

분석용 binary 사전 작성

```console
$ cd /usr/local/lib/mecab/dic/nkhandic
$ /usr/local/libexec/mecab/mecab-dict-index -f utf8 -t utf8
```

분석할 때 실제로 필요한 파일은 `char.bin`, `dicrc`, `matrix.bin`, `sys.dic`, `unk.dic`입니다.

## 사용법

### 실행시 사전을 지정

MeCab를 실행할 때 `-d` 옵션으로 사전 파일이 포함된 디렉토리를 지정할 수 있습니다.

```console
$ mecab -d /usr/local/lib/mecab/dic/nkhandic
```

위 방법으로는 실행할 때마다 사전을 지정할 필요가 있습니다.

### 설정 파일에 사전 경로 기술

홈 디렉토리에 `.mecabrc` 파일을 작성하여 `dicdir`에 NK-HanDic 사전 파일이 포함된 디렉토리 경로를 기술할 수 있습니다.

```text
dicdir = /usr/local/lib/mecab/dic/nkhandic
```

위 방법으로는 항상 NK-HanDic으로 분석하게 됩니다.

### 입력문

NK-HanDic은 UTF-8 인코딩된 텍스트를 입력하여 형태소 분석을 실행합니다.
입력할 때에는 완성형 한글(Hangul Syllables 영역의 문자)가 아니라 초성·중성·종성으로 분리한 첫가끝 코드(조합형, 한글 자모 영역의 문자)로 기술할 필요가 있습니다.
예를 들어 완성형 한글의 '몸'(U+BAB8)은 한글 자모 영역의 글자를 사용하여 'ㅁ'(U+1106), 'ㅗ'(U+1169), 'ㅁ'(U+11B7)으로 나누어서 입력으로 주어야 합니다.

이러한 변환 처리는 임의로 스크립트를 만들어서 처리해도 괜찮습니다.
이 프로젝트에서는 Perl 스크립트 `k2jamo.pl`과 Python 스크립트 `k2jamo.py`를 제공하고 있습니다. `tools` 디렉토리를 참조하십시오.

`k2jamo.pl`로 `input.txt`를 분석할 경우:

```console
$ perl k2jamo.pl input.txt | mecab -d /usr/local/lib/mecab/dic/nkhandic
```

혹은 문장을 직접 입력할 경우:

```console
$ echo "겨울 방학 때 뭐 했어요?" | perl k2jamo.pl | mecab -d /usr/local/lib/mecab/dic/nkhandic
```

처럼 처리할 수 있습니다.

## 품사 정보

품사 정보에 관한 정보는 [품사 정보](pos_detail.md) 문서를 참조하시기 바랍니다.

## 사전 구축과 등록 항목

### 사전 구축 방법

사전 구축에 있어서 MeCab의 재학습(再學習) 기능([オリジナル辞書/コーパスからのパラメータ推定(일본어)](https://taku910.github.io/mecab/learn.html) 참조)을 이용하였습니다.
현대 한국어 분석 사전 [HanDic](https://github.com/okikirmui/handic) 데이터에 아래에서 설명하는 조선어 항목(`z_NK.csv`)을 추가하여 HanDic 학습 모델을 바탕으로 조선어 데이터를 가지고 재학습하였습니다.
구체적인 절차는 다음과 같습니다.

```console
# mecab-dict-index 처리 후
# handic_model: HanDic 학습 모델, corpus.txt: 조선어 학습 데이터
# 모든 파일이 같은 디렉토리에 있다고 가정
# 재학습 실행
/usr/local/libexec/mecab/mecab-cost-train -p 2 -M handic_model -c 1.0 corpus.txt model
# 배포용 사전 작성(final 디렉토리에 출력할 경우)
/usr/local/libexec/mecab/mecab-dict-gen -o final -d . -m model
# 분석용 binary 사전 작성
cd final
/usr/local/libexec/mecab/mecab-dict-index -f utf8 -t utf8
```

재학습에 사용한 조선어 데이터는 아래와 같습니다(3,005 문장).

#### 로동신문

[홈페이지](http://www.rodong.rep.kp/ko/)의 "사회문화생활" 분야 및 "진전하는 조선" 분야의 기사 리스트에서 각일 맨 처음에 게시된 기사를 선택했습니다.
해당 기사가 너무 긴 경우에는 그 다음 기사를 선택했고, 당일 기사가 하나밖에 없는 경우에는 해당 기사를 선택했습니다.

##### 사회문화생활

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

##### 전진하는 조선

  1. [지방공업공장들의 골조공사 련이어 결속(2024/6/1, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0wMS1OMDAxQDRANEBAMEA0OA==)
  1. [검덕지구의 산악협곡도시건설을 추진(2024/6/2, 5면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0wMi1OMDA3QDRANEBAMEA0Ng==)
  1. [당의 대자연개조구상실현에서 이룩된 자랑찬 성과(2024/6/2, 5면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0wMy1OMDE3QDRAM0BAMEA0NQ==)
  1. [올해 계획된 농촌살림집건설에서 전국의 앞장에 섰다(2024/6/4, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0wNC1OMDAyQDRAM0BAMEA0Mw==)
  1. [대중운동의 불길속에 더욱 강해지는 소년단원들의 장한 대오(2024/6/6, 3면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0wNi1OMDA5QDRAM0BAMEA0Mg==)
  1. [철강재생산토대를 가일층 강화해나간다(2024/6/8, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0wOC1OMDAzQDRAM0BAMEA0MA==)
  1. [당의 지방공업발전정책관철을 위해 한결같이 떨쳐나섰다(2024/6/9, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0wOS1OMDAyQDRAM0BAMEAzOA==)
  1. [１２개 중요고지점령을 위한 투쟁의 전렬에서 기간공업부문이 기세차게 내달린다(2024/6/10, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0xMC1OMDAyQDRAM0BAMEAzNw==)
  1. [특파기자들이 보내온 소식(2024/6/11, 4면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0xMS1OMDE2QDRAM0BAMEAzNg==)
  1. [조선중앙통신사 보도 사회주의농촌진흥을 가속화하며 련이어 이룩되는 수리화의 변혁적성과(2024/6/12, 3면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0xMi1OMDEyQDRAM0BAMEAzNA==)
  1. [지방공업혁명의 전구마다에 결사관철의 기상 나래친다(2024/6/13, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0xMy1OMDA1QDRAM0BAMEAzMw==)
  1. [북방의 철의 기지가 철강재생산에서 장훈을 불렀다(2024/6/14, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0xNC1OMDAxQDRAM0BAMEAzMg==)
  1. [새 기준, 새 기록창조로 날이 밝고 해가 저문다(2024/6/15, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0xNS1OMDAxQDRAM0BAMEAzMQ==)
  1. [지방의 전면적진흥을 안아오기 위한 우리당 １０년목표실행의 돌파구 개척(2024/6/17, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0xNy1OMDAxQDRAMkBAMEAzMA==)
  1. [당정책의 생활력을 과시하며 황남의 대지에 전례없는 밀, 보리풍작이 들었다(2024/6/21, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0yMS1OMDAxQDRAMkBAMEAyOQ==)
  1. [사회주의전야에서 련일 전해지는 흐뭇한 밀, 보리수확소식(2024/6/24, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0yNC1OMDAzQDRAMkBAMEAyOA==)
  1. [새로운 건설신화를 끊임없이 창조해간다(2024/6/28, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0yOC1OMDAyQDRAMkBAMEAyNg==)

#### 조선어 교재

각 교재의 대화문이나 강독문을 사용했습니다.

  - 조선어기초(류학생용, 2017, 김형직사범대학)
  - 조선어1~5(류학생용, 2022, 김일성종합대학)

### 등록된 항목

사전 항목은 현대 한국어 분석 사전 [HanDic](https://github.com/okikirmui/handic)를 바탕으로 하여 추가로 다음과 같은 자료를 참조하여 등록하였습니다.

  - [우리말샘](https://opendict.korean.go.kr/): 2024년 4월 시점의 전체 데이터 중 '북한말'로 기술된 항목
  - 산업연구원DB [KIET 북한 산업·기업 DB](http://nkindustry.kiet.re.kr/comp/list.do): 북한 기업 목록
  - [통일부 북한정보포털](https://nkinfo.unikorea.go.kr/nkp/word/nkword.do): ‘북한 지도’에서 ‘백화점’, ‘상점’, ‘휴양소’, ‘호텔’, ‘려관’, ‘야영소’, ‘대학’ 등의 검색어로 검색된 항목
  - 통일부 북한정보포털에서 취득한 『북한 기관별 인명록(별책)』 2022에 기재된 기관명 일부
  - 현대 한국어 분석 사전 [HanDic](https://github.com/okikirmui/handic) 데이터 중 용언의 일부를 수정

『조선말대사전』(2017년)도 참조하여 추가한 항목도 있습니다.

## Author

  - Yoshinori Sugai(스가이 요시노리/須賀井義教, Kindai University)

## Copyrights

Copyright (c) 2024- Yoshinori Sugai. All rights reserved.

''NK-HanDic'' is under [BSD-3-Clause](https://opensource.org/licenses/BSD-3-Clause).
