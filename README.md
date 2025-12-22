# NK-HanDic: morphological analysis dictionary for North Korean language

[日本語Readme](README_ja.md)

NK-HanDic(북한딕)은 형태소 분석 엔진 [MeCab](https://taku910.github.io/mecab/)(메카부)로 북한의 언어(조선어)를 분석하기 위해 개발된 분석 사전입니다.
21만 개가 넘는 항목으로 구성되어 있고, 로동신문이나 북한의 조선어 교재 등 문어를 중심으로 한 데이터로 학습, 구축되었습니다.

저작권 문제로 학습 데이터 자체는 공개하지 않지만 모델 파일은 패키지에 포함되어 있습니다.

## Requirements

  - MeCab
  - Python or Perl

> ⚠️ **중요**
>
> NK-HanDic은 **완성형 한글을 입력으로 사용하지 않습니다.**  
> 형태소 분석을 하기 전에 **반드시 Jamo(자모) 단위로 분해된 문자열**을 입력으로 사용해야 합니다.
>
> 자모 변환을 거치지 않은 입력에 대해서는 정상적인 분석 결과를 보장하지 않습니다.

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

> ※ `mecab-dict-index`의 위치는 MeCab 설치 방식(Homebrew, apt, 소스 빌드 등)에 따라 다를 수 있습니다.  
> 명령어를 찾을 수 없는 경우 다음 명령으로 libexec 경로를 확인하십시오.
>
> ```bash
> mecab-config --libexecdir
> ```

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

### 토큰화 처리(tokenize)

출력 포맷을 지정하는 `-O` 옵션을 사용하여 토큰화 처리를 할 수 있습니다.
출력 포맷으로 `tokenize`를 지정합니다.

```console
$ echo "모든 치료조건과 환경이 그쯘하게 갖추어진 료양소의 구내에 야외휴식터를 번듯하게 꾸리고 수종이 좋은 나무들과 꽃관목들로 이채로운 원림경관을 조성하여 주변풍치를 한껏 돋구었다." | perl k2jamo.pl | mecab -d /usr/local/lib/mecab/dic/nkhandic -O tokenize
모든 치료 조건 과 환경 이 그쯘하 게 갖추어 지 ㄴ 료양소 의 구내 에 야외 휴식터 를 번듯 하 게 꾸리 고 수종 이 좋으 ㄴ 나무 들 과 꽃 관목 들 로 이채 로우 ㄴ 원림 경관 을 조성 하여 주변 풍치 를 한껏 돋구어 ㅆ 다 .
```

## 사용법(Python)

[PyPI](https://pypi.org/project/nkhandic/)에서 `nkhandic` 패키지를 공개했습니다.
`mecab-python3` 패키지와 입력문을 변환하기 위한 `jamotools` 등의 패키지와 함께 사용합니다.

> PyPI로 배포되는 `nkhandic` 패키지에는 **이미 빌드된 사전 디렉터리**가 포함되어 있으며, 별도로 사전을 구축할 필요가 없습니다.

설치:

```bash
pip install nkhandic mecab-python3 jamotools
```

예:

```Python
import MeCab
import nkhandic
import jamotools

mecaboption = f'-r /dev/null -d {nkhandic.DICDIR}'

tokenizer = MeCab.Tagger(mecaboption)
tokenizer.parse('')

# 로동신문 2024년 5월 1일자 사설
sentence = u'경애하는 총비서동지에 대한 절대적인 충성심을 지니고 당중앙의 구상과 결심을 철저한 실천행동으로 받들어나가야 한다.'

# ⚠️ 반드시 Jamo(자모)로 변환한 문자열을 입력으로 사용해야 함
jamo = jamotools.split_syllables(sentence, jamo_type="JAMO")

node = tokenizer.parseToNode(jamo)
while node:
    print(node.surface, node.feature)
    node = node.next
```


## 품사 정보

품사 정보에 관한 정보는 [품사 정보](docs/pos_detail.md) 문서를 참조하시기 바랍니다.

## 사전 구축과 등록 항목

### 사전 구축 방법

사전 구축에 있어서 MeCab의 재학습(再學習) 기능([オリジナル辞書/コーパスからのパラメータ推定(일본어)](https://taku910.github.io/mecab/learn.html) 참조)을 이용하였습니다.
현대 한국어 분석 사전 [HanDic](https://github.com/okikirmui/handic) 데이터에 아래에서 설명하는 조선어 항목(`z_NK.csv`)을 추가하여 HanDic 학습 모델을 바탕으로 조선어 데이터를 가지고 재학습하였습니다.
구체적인 절차는 다음과 같습니다.

mecab-dict-index 처리 후, 재학습 실행. 모든 파일이 같은 디렉토리에 있는 경우.
`handic_model`: HanDic 학습 모델, `corpus.txt`: 조선어 학습 데이터

```console
$ /usr/local/libexec/mecab/mecab-cost-train -p 2 -M handic_model -c 1.0 corpus.txt model
$ /usr/local/libexec/mecab/mecab-dict-gen -o final -d . -m model
$ cd final
$ /usr/local/libexec/mecab/mecab-dict-index -f utf8 -t utf8
```

### 학습용 데이터

재학습에 사용한 조선어 데이터 현황은 아래와 같습니다.

| 자료   |   분야      | 기사/권 수 | 문장 수 |
|------------|---------|-------:|-----------:|
| 로동신문     | 사회문화생활  |     24|      428|
|            | 전진하는 조선  |     17|     538|
|            | 사설 |     18|     325|
|            | 날짜별 기사    |     215 |    3,534|
| 조선어 교재  |         |    24|           11,782|
| 합계 |         |     |     16,607|

자세한 목록은 [학습용 데이터](docs/data_list.md) 문서를 참조할 것.

### 등록된 항목

사전 항목은 현대 한국어 분석 사전 [HanDic](https://github.com/okikirmui/handic)를 바탕으로 하여 추가로 다음과 같은 자료를 참조하여 등록하였습니다.

  - [우리말샘](https://opendict.korean.go.kr/): 2024년 4월 시점의 전체 데이터 중 '북한말'로 기술된 항목
  - 산업연구원DB [KIET 북한 산업·기업 DB](http://nkindustry.kiet.re.kr/comp/list.do): 북한 기업 목록
  - [통일부 북한정보포털](https://nkinfo.unikorea.go.kr/nkp/word/nkword.do): ‘북한 지도’에서 ‘백화점’, ‘상점’, ‘휴양소’, ‘호텔’, ‘려관’, ‘야영소’, ‘대학’ 등의 검색어로 검색된 항목
  - 통일부 북한정보포털에서 취득한 『북한 기관별 인명록(본권)』『북한 기관별 인명록(별책)』 2022, 2023에 기재된 기관명, 인명의 일부
  - 현대 한국어 분석 사전 [HanDic](https://github.com/okikirmui/handic) 데이터 중 용언의 일부를 수정

『조선말대사전』(2017년)을 참조하여 추가한 항목도 있습니다.

## Author

  - Yoshinori Sugai(스가이 요시노리/須賀井義教, Kindai University)

## Reference

  - 스가이 요시노리(2024), [`북한 조선어 형태소 분석 사전 구축에 관한 연구'](http://doi.org/10.33641/kolex.2024..44.33), 한국사전학 제44호, 서울: 한국사전학회, pp.33-63.

## Copyrights

Copyright (c) 2024- Yoshinori Sugai. All rights reserved.

''NK-HanDic'' is under BSD-3-Clause.
