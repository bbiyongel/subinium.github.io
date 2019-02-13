---
title : \[ML with Python\] 3장 비지도 학습과 데이터 전처리 - 차원 축소, 특성 추출, 매니폴드 학습
category :
  - ML
tag :
  - python
  - deep-learning
  - AI
  - machine learning
  - 머신러닝
  - 비지도 학습
  - 데이터 전처리
  - 입문
  - subinium
  - 소스코드

sidebar:
  nav: sidebar-MLwithPython

use_math : true

header:
  teaser : /assets/images/category/ml.jpg
  overlay_image : /assets/images/category/ml.jpg
published : false
---

3.4 차원 축소, 특성 추출, 매니폴드 학습

본 문서는 [파이썬 라이브러리를 활용한 머신러닝] 책을 기반으로 하고 있으며, subinium(본인)이 정리하고 추가한 내용입니다. 생략된 부분과 추가된 부분이 있으니 추가/수정하면 좋을 것 같은 부분은 댓글로 이야기해주시면 감사하겠습니다.

## 3.4.0 INTRO

데이터 변환의 이유는 여러 가지입니다.
가장 일반적인 동기는 시각화하거나, 데이터를 압축하거나, 추가적인 처리를 위해(지도 학습) 정보가 더 잘 드러나는 표현을 찾기 위해서입니다.
이번 절에서는 다음과 같은 알고리즘을 소개합니다.

- 주성분 분석 : 가장 간단하고 흔히 사용
- 비음수 행렬 분석 : 특성 추출
- t-SNE : 2차원 산점도를 이용해 시각화 용도로 사용

## 3.4.1 주성분 분석(PCA)

**주성분 분석(principal component analysis, PCA)** 은 통계적으로 상관관계가 없도록 데이터셋을 회전시기는 기술입니다.
단순히 회전만 하는 것은 아니고 회전을 한 뒤에 데이터를 설명하는데 필요한 (새로운)특성 중 일부만 선택합니다.
다음 예제로 그 의미를 확실히 할 수 있습니다.

![3](https://i.imgur.com/dRAtKnq.png)

첫 번째 그래프는 원본 데이터 포인트를 색으로 구분해 표시한 것입니다. 이 알고리즘은 먼저 **성분 1** 이라고 쓰인 분산이 가장 큰 방향을 찾습니다.
이 방향이 데이터에서 가장 많은 정보를 담고 있는 방향입니다. 특성들의 상관관계가 큰 방향입니다.
그리고 첫 번째 방향과 직각인 방향 중에서 가장 많은 정보를 담은 방향을 찾습니다.
2차원에서는 하나지만 고차원에서는 무한히 많은 직각 방향이 있습니다.

화살표의 방향은 중요하지 않습니다. 반대로 그려도 괜찮다는 것입니다. :-)
그렇게 찾은 방향을 데이터에 있는 주된 분산 방향이라고 하여 **주성분(principal component)** 라고 합니다. 일반적으로 원본 특성 개수만큼의 주성분이 있습니다. (기저 벡터의 개념으로 생각하면 됩니다.)

두 번째 그래프는 같은 데이터지만 주성분 1과 2를 각각 x축과 y축에 나란하도록 회전한 것입니다. 회전하기 전에 데이터에서 평균을 빼서 원점에 맞췄습니다. PCA에 의해 회전된 두 축은 연관되어 있지 않으므로 변환된 데이터에서 상관관계 행렬이 대각선 방향을 제외하고는 0이 됩니다.

PCA는 주성분의 일부만 남기는 차원 축소 용도로 사용할 수 있습니다. 이 예에서는 왼쪽 아래 그림과 같이 첫 번째 주성분만 유지하려고 합니다. 그렇게 되면 2차원 -> 1차원으로 차원이 감소합니다. 가장 유용한 성분을 남기는 것입니다.

마지막으로 데이터에 다시 평균을 더하고 반대로 회전시킵니다. 이 결과가 마지막 그래프입니다.
데이터 포인터들은 원래 특성 공간에 놓여 있지만 첫 번째 주성분의 정보만 담고 있습니다.
이 변환은 ***데이터에서 노이즈를 제거하거나 주성분에서 유지되는 정보를 시각화*** 하는 데 종종 사용합니다.

### PCA를 적용해 유방암 데이터셋 시각화하기

PCA가 가장 널리 사용되는 분야는 고차원 데이터셋의 시각화입니다.
세 개 이상의 특성을 가진 데이터를 산점도로 표현하는 것은 어렵습니다.
cancer 데이터셋에는 특성을 30개나 가지고 있어, 산점도를 그리기 위해서는 435개의 산점도를 그려야합니다. ($_{30}C_{2}$)
이보다 쉬운 방법은 양성과 악성 두 클래스에 대해 각 특성의 히스토그램을 그리는 것입니다.

![4](https://i.imgur.com/DwrZtc3.png)

이 그림은 각 특성에 대한 히스토그램으로 특정 간격(bin)에 얼마나 많은 데이터 포인트가 나타나는지 횟수를 센 것입니다.
각 그래프는 히스토그램 두 개를 겹쳐놓은 것으로 초록색은 양성 클래스의 포인트, 푸른색은 음성 클래스의 포인트입니다.

그래프를 보면 어떤 특성이 차이가 나고, 어떤 특성이 영향이 큰지 정도는 감이 옵니다.
하지만 특성 간의 상호작용이나 이 상호작용이 클래스와 어떤 관련이 있는지는 전혀 알려주지 못합니다.
PCA를 이용하면 주요 상호작용을 찾아낼 수 있어 더 나은 그림을 만들 수 있습니다.

이제 스케일을 조정하고 PCA를 활용해 산점도를 그려보겠습니다.
우선 `StandardScaler`를 사용해 각 특성의 분산이 1이 되도록 조정하겠습니다.

``` python
from sklearn.datasets import load_breast_cancer
from sklearn.preprocessing import StandardScaler
cancer = load_breast_cancer()

scaler = StandardScaler()
scaler.fit(cancer.data)
X_scaled = scaler.transform(cancer.data)
```

PCA 변환을 학습하고 적용하는 것은 전처리만큼 간단합니다.

1. PCA 객체 생성
2. `fit` 메서드를 호출해 주성분 찾기
3. `transform` 메서드를 호출해 데이터 호출시키고 차원 축소하기

기본값일 때는 회전과 이동만 하니, 차원을 줄이려면 매개변수로 전달해야 합니다.

``` python
from sklearn.decomposition import PCA
# 데이터의 처음 두 개 주성분만 유지시킵니다
pca = PCA(n_components=2)
# 유방암 데이터로 PCA 모델을 만듭니다
pca.fit(X_scaled)

# 처음 두 개의 주성분을 사용해 데이터를 변환합니다
X_pca = pca.transform(X_scaled)
print("원본 데이터 형태: {}".format(str(X_scaled.shape)))
print("축소된 데이터 형태: {}".format(str(X_pca.shape)))

# 원본 데이터 형태: (569, 30)
# 축소된 데이터 형태: (569, 2)
```

그리고 맨 처음 두 개의 주성분을 이용해 산점도를 그립니다.

``` python
# 클래스를 색깔로 구분하여 처음 두 개의 주성분을 그래프로 나타냅니다.
plt.figure(figsize=(8, 8))
mglearn.discrete_scatter(X_pca[:, 0], X_pca[:, 1], cancer.target)
plt.legend(["negative", "positive"], loc="best")
plt.gca().set_aspect("equal")
plt.xlabel("1st pc")
plt.ylabel("2nd pc")
```

![5](https://i.imgur.com/WnAafLV.png)

PCA는 비지도 학습이므로 회전축을 찾을 때 어떤 클래스 정보도 사용하지 않습니다. 단순히 데이터에 있는 상관관계만 고려합니다.
두 클래스가 2차원 공간에서 꽤 잘 구분되는 것을 볼 수 있습니다.
이런 그림이면 선형 분류기로도 두 클래스를 잘 구분할 수 있을 것 같습니다.

하지만 PCA의 단점 중 하나는 그래프의 두 축을 해석하기 쉽지 않다는 점입니다.
여러 특성이 조합된 형태인 것만 알고있습니다. 이에 대한 정보는 PCA 객체가 학습될때(`fit` 메서드가 호출될 때입니다.) `components_` 속성에 저장됩니다.

``` python
print("PCA 주성분 형태: {}".format(pca.components_.shape))
# PCA 주성분 형태: (2, 30)
```

`components_`의 각 행은 주성분 하나씩을 나타내며 중요도에 따라 정렬되어 있습니다.
열은 원본 데이터의 특성에 대응하는 값입니다.

``` python
print("PCA 주성분: {}".format(pca.components_))
# PCA 주성분: [[ 0.21890244  0.10372458  0.22753729  0.22099499  0.14258969  0.23928535
#    0.25840048  0.26085376  0.13816696  0.06436335  0.20597878  0.01742803
#    0.21132592  0.20286964  0.01453145  0.17039345  0.15358979  0.1834174
#    0.04249842  0.10256832  0.22799663  0.10446933  0.23663968  0.22487053
#    0.12795256  0.21009588  0.22876753  0.25088597  0.12290456  0.13178394]
#  [-0.23385713 -0.05970609 -0.21518136 -0.23107671  0.18611302  0.15189161
#    0.06016536 -0.0347675   0.19034877  0.36657547 -0.10555215  0.08997968
#   -0.08945723 -0.15229263  0.20443045  0.2327159   0.19720728  0.13032156
#    0.183848    0.28009203 -0.21986638 -0.0454673  -0.19987843 -0.21935186
#    0.17230435  0.14359317  0.09796411 -0.00825724  0.14188335  0.27533947]]
```

이를 히트맵으로 시각화하면 이해하는데 용이합니다.

``` python
plt.matshow(pca.components_, cmap='viridis')
plt.yticks([0, 1], ["lst pc", "2nd pc"])
plt.colorbar()
plt.xticks(range(len(cancer.feature_names)), cancer.feature_names, rotation=60, ha='left')
plt.xlabel("feature")
plt.ylabel("pc")
plt.matshow(pca.components_, cmap='viridis')
plt.yticks([0, 1], ["lst pc", "2nd pc"])
plt.colorbar()
plt.xticks(range(len(cancer.feature_names)), cancer.feature_names, rotation=60, ha='left')
plt.xlabel("feature")
plt.ylabel("pc")
```

![6](https://i.imgur.com/QRAxEQU.png)

### 고유얼굴(eigenface) 특성 추출

PCA는 특성 추출에도 이용합니다. 원본

## 3.4.2 비음수 행렬 분해(NMF)

## 3.4.3 t-SNE를 이용한 매니폴드 학습