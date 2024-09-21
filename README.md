# NK-HanDic: morphological analysis dictionary for North Korean language

[日本語Readme](README_ja.md)

NK-HanDic(북한딕)은 형태소 분석 엔진 [MeCab](https://taku910.github.io/mecab/)(메카부)로 북한의 언어(조선어)를 분석하기 위해 개발된 분석 사전입니다.
21만 개가 넘는 항목으로 구성되어 있고, 로동신문이나 북한의 조선어 교재 등 문어를 중심으로 한 데이터로 학습, 구축되었습니다.

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

### 토큰화 처리(tokenize)

출력 포맷을 지정하는 `-O` 옵션을 사용하여 토큰화 처리를 할 수 있습니다.
출력 포맷으로 `tokenize`를 지정합니다.

```console
$ echo "모든 치료조건과 환경이 그쯘하게 갖추어진 료양소의 구내에 야외휴식터를 번듯하게 꾸리고 수종이 좋은 나무들과 꽃관목들로 이채로운 원림경관을 조성하여 주변풍치를 한껏 돋구었다." | perl k2jamo.pl | mecab -d /usr/local/lib/mecab/dic/nkhandic -O tokenize
모든 치료 조건 과 환경 이 그쯘하 게 갖추어 지 ㄴ 료양소 의 구내 에 야외 휴식터 를 번듯 하 게 꾸리 고 수종 이 좋으 ㄴ 나무 들 과 꽃 관목 들 로 이채 로우 ㄴ 원림 경관 을 조성 하여 주변 풍치 를 한껏 돋구어 ㅆ 다 .
```

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
$ /usr/local/libexec/mecab/mecab-cost-train -p 2 -M handic_model -c 1.0 corpus.txt model
# 배포용 사전 작성(final 디렉토리에 출력할 경우)
$ /usr/local/libexec/mecab/mecab-dict-gen -o final -d . -m model
# 분석용 binary 사전 작성
$ cd final
$ /usr/local/libexec/mecab/mecab-dict-index -f utf8 -t utf8
```

재학습에 사용한 조선어 데이터 현황은 아래와 같습니다.

| 자료   |   분야      | 기사/권 수 | 문장 수 |
|------------|---------|-------:|-----------:|
| 로동신문     | 사회문화생활  |     24|      428|
|            | 전진하는 조선  |     17|     538|
|            | 사설 |     18|     325|
| 조선어 교재  |         |    5|           2,042|
| 잡지 "조선문학" |  단편소설   |    4 |     200 |
| 합계 |         |     |     3,533|

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

##### 사설

각 기사가 너무 길기 때문에 무작위로 20개(일부는 15개) 문장씩 추출하여 학습 데이터로 삼았습니다.

1. [위대한 당의 령도따라 근로하는 인민의 나라, 사회주의조국을 애국적헌신으로 더욱 빛내이자(2024/5/1, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0wMS1OMDAxQDExQDFA7IKs7ISkQDBAMjM==)
  1. [일군들은 뚜렷한 목표를 가지고 사업을 전망적으로, 발전지향적으로 전개하자(2024/5/5, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0wNS1OMDAxQDExQDFA7IKs7ISkQDBAMjI==)
  1. [모든 력량을 총동원, 총집중하여 모내기를 제철에 질적으로 끝내자(2024/5/7, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0wNy1OMDAyQDExQDFA7IKs7ISkQDBAMjE==)
  1. [청년들은 당의 품속에서 백배해진 담력과 배짱으로 혁명전위의 영예와 존엄을 더 높이 떨쳐나가자(2024/5/19, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0xOS1OMDAxQDExQDFA7IKs7ISkQDBAMjA==)
  1. [당대회결정관철의 승산을 확정지어야 하는 올해의 상반년사업을 떳떳이 총화받자(2024/5/26, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0yNi1OMDAxQDExQDFA7IKs7ISkQDBAMTk==)
  1. [당의 지방발전정책실행으로 지역의 ３대혁명화를 추동하자(2024/5/28, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNS0yOC1OMDAxQDExQDFA7IKs7ISkQDBAMTg==)
  1. [련대적, 집단적혁신을 일으키자(2024/6/8, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0wOC1OMDAxQDExQDBA7IKs7ISkQDBAMTU==)
  1. [건당위업의 개척세대가 지녔던 리상과 신념, 정신을 본받자(2024/6/11, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0xMS1OMDAxQDExQDBA7IKs7ISkQDBAMTQ==)
  1. [로씨야련방 대통령 울라지미르 뿌찐동지를 열렬히 환영한다(2024/6/18, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0xOC1OMDAxQDExQDBA7IKs7ISkQDBAMTM==)
  1. [우리 당의 창건과 강화발전의 력사를 깊이 체득하자(2024/6/22, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0yMi1OMDAxQDExQDBA7IKs7ISkQDBAMTI==)
  1. [１９５０년대 조국수호정신을 필승의 무기로 틀어쥐고 우리 국가의 존엄과 국익을 억척같이 수호하자(2024/6/25, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNi0yNS1OMDA0QDExQDBA7IKs7ISkQDBAMTE==)
  1. [전당, 전민이 일심분발하여 당대회강령실현의 관건적해인 올해를 자랑찬 변혁적성과로 빛내이자(2024/7/4, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNy0wNC1OMDAxQDExQDBA7IKs7ISkQDBAMTA==)
  1. [드높은 자신심과 배가된 분발력으로 사회주의건설의 상승국면을 계속혁신, 련속도약에로 이어나가자(2024/7/6, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNy0wNi1OMDAxQDExQDBA7IKs7ISkQDBAOQ==)
  1. [일군들은 ５개년계획완수의 확정적담보를 마련하기 위한 투쟁에서 비상한 투신력과 완강한 집행력을 발휘하자(2024/7/11, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNy0xMS1OMDAxQDExQDBA7IKs7ISkQDBAOA==)
  1. [당조직들의 역할에 인민의 생명안전과 국가의 부흥이 달려있다(2024/7/17, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNy0xNy1OMDAxQDExQDBA7IKs7ISkQDBANg==)
  1. [일군들은 사업방법과 작풍을 결정적으로 개선하여 당과 국가앞에 지닌 책무를 다하자(2024/7/21, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wNy0yMS1OMDAxQDExQDBA7IKs7ISkQDBANQ==)
  1. [당의 령도따라 용기백배, 신심드높이 떨쳐일어나 큰물피해지역에 사회주의락원을 보란듯이 펼쳐놓자(2024/8/1, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wOC0wMS1OMDAxQDExQDBA7IKs7ISkQDBAMw==)
  1. [당원들이여, 당중앙의 부름을 받들고 전화위복의 기적을 창조하는 투쟁에서 선봉적역할을 다하자(2024/8/2, 1면)](http://www.rodong.rep.kp/ko/index.php?MTJAMjAyNC0wOC0wMi1OMDAxQDExQDBA7IKs7ISkQDBAMg==)

#### 조선어 교재

각 교재의 대화문이나 강독문을 사용했습니다.

  - 조선어기초(류학생용, 2017, 김형직사범대학): 제1과~제25과 회화문만 입력.
  - 조선어2~5(류학생용, 2022, 김일성종합대학): 2/3권의 '1. 본문', 4/5권의 '대화본문' 및 '강독본문'만 입력. 

#### 소설

잡지 "조선문학"(발행: 문학예술출판사)에 수록된 단편소설 중, 각 호 맨 처음에 실린 것을 선택했습니다.
각 소설 앞 부분 50개 문장씩 추출하여 데이터로 삼았습니다.

  1. 비단숲 설레인다(변일문, 조선문학 주체112(2023)년 제1호, pp.7-25)
  1. 영생의 순간(김경화, 조선문학 주체112(2023)년 제2호, pp.7-26)
  1. 아버지(김동호, 조선문학 주체112(2023)년 제3호, pp.9-20)
  1. 보화의 산악(라광철, 조선문학 주체112(2023)년 제4호, pp.9-24)

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

## Copyrights

Copyright (c) 2024- Yoshinori Sugai. All rights reserved.

''NK-HanDic'' is under [BSD-3-Clause](https://opensource.org/licenses/BSD-3-Clause).
