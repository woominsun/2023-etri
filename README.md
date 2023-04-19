# 2023 제2회 ETRI 휴먼이해 인공지능 논문경진대회
>본 대회는 한국전자통신연구원(ETRI)이 주최하고 과학기술정보통신부와 국가과학기술연구회(NST)가 후원합니다.


- 멀티모달 감정 데이터셋 활용 감정인식 기술 분야 [바로가기](https://aifactory.space/competition/detail/2234)
- KEMDy19 (성우 대상 상황극) 데이터셋 [바로가기](https://nanum.etri.re.kr/share/kjnoh/KEMDy19?lang=ko_KR)

## Code
`audio CNN.py` : 음성 신호 감정표현 모델

`eda XGBClassifier bf.py` : EDA 감정반응+감정표현 모델

`eda XGBClassifier now.py` : EDA 감정표현 모델

`text KoBERT bf.py` : 텍스트 감정반응+감정표현 모델

`soft voting.py` : 최종 멀티모달


# 논문 주제
## Abstract
 최근 인공지능 기술의 발전으로 대화에서의 감정인식 기술에 대한 연구가 지속적으로 이루어지고 있다. 기존 연구에서는 단일 모달보다 멀티 모달 모델을 사용하여 감정을 분류하는 것에 초점을 맞추고 있지만, 더 나아가 대화에서의 문맥이나 감정 반응을 고려해 발화자의 감정을 인식하는 방법에 대해 제안하고자 한다. ‘KEMDy19’ 데이터셋을 사용하여 감정을 분류한 결과 감정반응을 적용한 모델의 성능이 적용하지 않은 모델 성능보다 약 0.2%-10.1% 정도 향상되었고, 단일 모달을 사용했을 때 보다 멀티 모달 모델을 기반으로 감정을 분류했을 때 향상된 성능을 보였다.

## 감정 반응과 감정 표현
![image](https://user-images.githubusercontent.com/130694680/232826978-8df6d142-e1a8-4709-9178-8fcd7838fbb8.png)

- 감정 반응(Emotion Response, {})

  두 사람의 대화에서 상대방의 발화를 듣고 내부적으로 생성되는 감정적 반응

- 감정 표현(Emotion Expression, [])

  발화자가 발화할 때 언어적 표현, 비언어적 행동을 통해 감정을 외부적으로 표현

## 모델 아키텍처
![image](https://user-images.githubusercontent.com/130694680/232827002-4ca6952e-ca99-4d11-9358-44b8c03b31dc.png)

### Data shape
- Text

```
X_train (8721, 9) y_train (8721, )

X_test (2230, 9) y_test (2230, )
```
하나의 발화 세그먼트에서 2개 이상의 감정 레이블이 존재하는 경우, 감정 레이블 개수만큼 발화 세그먼트를 중복 생성하여 발화 세그먼트 당 단일 감정 레이블로 변환하였다. 총 10,956개의 데이터를 구성하였으며, 감정 레이블은 있지만 텍스트가 없는 데이터 4개와 텍스트는 있지만 감정 레이블이 없는 데이터 1개를 삭제하여 총 10,951개의 데이터셋을 구성하였다. 감정반응 반영을 위해 감정반응과 텍스트를 결합시켜 감정반응을 포함한 텍스트를 생성하였으며 이후, KoBERT 모델의 토큰화를 통해 전처리한 데이터를 KoBERT 모델에 입력하여 감정을 분류하였다.
- EDA

```
X_train (5968, 172) y_train (5968, )

X_test (2021, 172) y_test (2021, )
```
주기에 따라 측정된 EDA 데이터는 발화 세그먼트 별로 데이터 shape이 동일하지 않아 zero padding을 이용해 가장 길게 측정된 EDA 데이터의 shape으로 맞추었다. 이로 인해 실제 측정된 데이터가 아닌 0으로 입력된 데이터가 존재하게 되어 분석의 정확도를 향상시키기 위해 0으로 입력된 데이터의 개수를 줄이고자 하였다. 발화 세그먼트 당 가장 길게 측정된 EDA의 측정 횟수는 172번이었고, 그 중 측정 횟수가 86번 이상인 발화 세그먼트가 약 1.68%만 존재함에 따라 실제 측정된 데이터의 측정 횟수가 86번 이상인 발화 세그먼트 데이터를 제거하여 총 7,989개의 데이터로 구성하였다. 발화자의 발화 이전 EDA 데이터를 발화 데이터에 연결하여 감정반응을 반영한 데이터셋을 생성하고 최적의 파라미터를 적용한 XGBClassifier에 입력하여 감정을 분류하였다.
- Audio

```
X_train (9360, 128, 3300) y_train (9360, 7)

X_test (2095, 128, 3300) y_test (2095, 7)
```
Train 음성 데이터의 95%가 14초 이하의 길이로 데이터 불균형을 보완하기 위해 14초를 초과하는 음성 데이터에 대해 음성 길이의 평균값인 10초를 기준으로 5초 간격의 sliding window를 적용하여 총 1,665개의 동일한 발화 세그먼트의 음성 데이터를 생성하였다. Test 데이터에 대해서도 Train 데이터와 동일한 방법을 적용하여 전체 음성 길이 분포에서 상위 5%에 해당하는 데이터를 데이터 평균 길이인 10초의 음성 파일로 변환하였다.
음성신호는 데이터 특성상 감정표현 정보만 담고 있으므로 감정반응을 적용한 데이터셋을 구성하지 않았다. 이후MFCC(Mel-Frequency Cepstral Coefficient)를 이용하여 음성 데이터의 특징 값을 추출하였고, 데이터의 최대 길이를 기반으로 패딩 작업을 하였다. 그림 3과 같이 CNN 모델 구조를 구성하여 감정 분류 모델로 활용하였다.

## 모델 성능 평가
![image](https://user-images.githubusercontent.com/130694680/231962041-547ca899-2c50-4076-830b-76a94f454bca.png)

## 실험 결과
![image](https://user-images.githubusercontent.com/130694680/231944849-ae8b71ed-e63b-4ce7-9541-3c38dc31247e.png)
