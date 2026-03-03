# NK-HanDic: morphological analysis dictionary for North Korean language

日本語Readme: [README_ja.md](README_ja.md)

NK-HanDic(북한딕)은 형태소 분석 엔진 [MeCab](https://taku910.github.io/mecab/)(메카부)로 북한의 언어(조선어)를 분석하기 위해 개발된 분석 사전입니다.
21만 개가 넘는 항목으로 구성되어 있고, 로동신문이나 북한 소설 등 문어를 중심으로 한 데이터로 학습, 구축되었습니다.

저작권 문제로 학습 데이터 자체는 공개하지 않지만 모델 파일은 패키지에 포함되어 있습니다.


## Quick Start(Python / pip)

Python 패키지 **`nkhandic`** 을 사용하면 **사전 파일 구축 없이 바로 형태소 분석을 시작**할 수 있습니다. 

### 패키지 설치

```bash
pip install nkhandic mecab-python3 jamotools
```

### 형태소 분석 예제

```python
import nkhandic

text = "량강도안의 혁명사적부문 강사들이 일군들과 당원들과 근로자들의 투쟁열의를 분출시키는 경제선동활동을 활발히 벌리고있다."

print(nkhandic.pos_tag(text))
print(nkhandic.tokenize_hangul(text, mode="surface"))
print(nkhandic.convert_text_to_hanja_hangul(text))
```

**출력 결과**

```python
['량강도', '안01', '의10', '혁명사', '적18', '부문06', '강사04', '들09', '이25', '일군002', '들09', '과12', '당원03', '들09', '과12', '근로자', '들09', '의10', '투쟁', '열의', '를', '분출03', '시키다02', '는03', '경제선동001', '활동02', '을02', '활발히', '벌리다004', '고25', '있다01', '다06', '.']
[('량강도', 'NNP'), ('안01', 'NNG'), ('의10', 'JKG'), ('혁명사', 'NNG'), ('적18', 'XSN'), ('부문06', 'NNG'), ('강사04', 'NNG'), ('들09', 'XSN'), ('이25', 'JKS'), ('일군002', 'NNG'), ('들09', 'XSN'), ('과12', 'JC'), ('당원03', 'NNG'), ('들09', 'XSN'), ('과12', 'JC'), ('근로자', 'NNG'), ('들09', 'XSN'), ('의10', 'JKG'), ('투쟁', 'NNG'), ('열의', 'NNG'), ('를', 'JKO'), ('분출03', 'NNG'), ('시키다02', 'XSV'), ('는03', 'ETM'), ('경제선동001', 'NNG'), ('활동02', 'NNG'), ('을02', 'JKO'), ('활발히', 'MAG'), ('벌리다004', 'VV'), ('고25', 'EC'), ('있다01', 'VX'), ('다06', 'EF'), ('.', 'SF')]
兩江道안의 革命史的部門 講師들이 일군들과 黨員들과 勤勞者들의 鬪爭熱意를 噴出시키는 經濟煽動活動을 活潑히 벌리고있다.
```

### Python 패키지 (`nkhandic`)

- PyPI: https://pypi.org/project/nkhandic/
- MeCab Python 래퍼(`mecab-python3`), 한글 처리 페키지(`jamotools`)와 함께 사용

## Dictionary Build

Python을 사용하지 않을 경우, 로컬에서 MeCab으로 형태소 분석 처리를 할 경우에 해당합니다.

## Requirements

  - MeCab
  - Python or Perl

> ⚠️ **중요**
>
> NK-HanDic은 **완성형 한글을 입력으로 사용하지 않습니다.**  
> 형태소 분석을 하기 전에 **반드시 Jamo(자모) 단위로 분해된 문자열**을 입력으로 사용해야 합니다.
>
> 자모 변환을 거치지 않은 입력에 대해서는 정상적인 분석 결과를 보장하지 않습니다.

### 사전 빌드 절차(요약)

`mecab-dict-index`, `mecab-dict-gen` 등의 위치는 `mecab-config --libexecdir`의 출력을 참조하십시오.
아래에서는 `/usr/local/libexec/mecab`에 있다고 가정.

```bash
# git clone
git clone https://github.com/okikirmui/nkhandic.git
cd handic
# 색인
/usr/local/libexec/mecab/mecab-dict-index -f utf8 -t utf8
# 기학습 모델 파일 model을 사용하여 binary 사전 구축
# /usr/local/lib/mecab/dic/nkhandic 디렉토리에 출력
/usr/local/libexec/mecab/mecab-dict-gen -o /usr/local/lib/mecab/dic/nkhandic -m model
# 배포용 사전 구축
$ cd /usr/local/lib/mecab/dic/nkhandic
$ /usr/local/libexec/mecab/mecab-dict-index -f utf8 -t utf8
```

분석할 때 실제로 필요한 파일은 `char.bin`, `dicrc`, `matrix.bin`, `sys.dic`, `unk.dic`입니다.


## Usage

### 실행시 사전을 지정

MeCab를 실행할 때 `-d` 옵션으로 사전 파일이 포함된 디렉토리를 지정할 수 있습니다.

```Shell
$ mecab -d /usr/local/lib/mecab/dic/nkhandic
```

위 방법으로는 실행할 때마다 사전을 지정할 필요가 있습니다.

### 설정 파일에 사전 경로 기술

홈 디렉토리에 `.mecabrc` 파일을 작성하여 `dicdir`에 HanDic 사전 파일이 포함된 디렉토리 경로를 기술할 수 있습니다.

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

```Shell
$ perl k2jamo.pl input.txt | mecab -d /usr/local/lib/mecab/dic/nkhandic
```

혹은 문장을 직접 입력할 경우:

```Shell
$ echo "겨울 방학 때 뭐 했어요?" | perl k2jamo.pl | mecab -d /usr/local/lib/mecab/dic/nkhandic
```

처럼 처리할 수 있습니다.

### 토큰화 처리(tokenize)

출력 포맷을 지정하는 `-O` 옵션을 사용하여 토큰화 처리를 할 수 있습니다.
출력 포맷으로 `tokenize`를 지정합니다.

```console
$ echo "경애하는 김정은동지께서는 다음과 같이 말씀하시였다." | perl k2jamo.pl | mecab -d /usr/local/lib/mecab/dic/nkhandic -O tokenize
경애 하 는 김 정은 동지 께서 는 다음 과 같이 말씀 하 시여 ㅆ 다 .
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

| 자료    |  분야  |  기사/권 수  |  문장 수  |
|--------|-------|-----------:|---------:|
| 로동신문 | 사설 | 18 | 325 |
| 로동신문 | 날짜별 기사 | 293 | 5,059 |
| 로동신문 | 사회문화생활 | 24 | 428 |
| 로동신문 | 전진하는 조선 | 17 | 538 |
| 조선문학 | 소설 | 8 | 400 |
| **합계** |   | **360** | **6,750** |

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
