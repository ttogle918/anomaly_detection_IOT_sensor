# Anomoly_detection ( 전류계 )


## 데이터셋 분석

input : 모터에 부착된 전류 센서에서 4kHz로 측정된 전류 정보 (R, S, T상 전류) 값, Train Set에는 정상/고장에 대한 레이블(label) 정보 없음

wavelength : 시간
x : R성 전류
y : S성 전류
z : T성 전류

output : 정상 / 고장 혹은 정상/고장일 확률

label 정보가 없기 때문에 비지도학습으로 output을 도출해야한다.

-------------------------

중성선이 0에 가까울수록 정상일 확률이 높고, 불평형률이 30% 이상이라면 고장날 확률이 높다.

이 기준으로 고장났거나 고장날 확률이 높은지 분류하기 위해 각 전류에 위상차를 표현하기 위해 위와 같은 계산을 하였다.

그런데 대부분의 데이터에서 0과 가깝게 나오지 않았다.

하지만 단순히 x+y+z를 하면 0과 근사한 값이 나온다.

(사실, A로 나타나려면 음수가 나올 수가 없다...)

따라서 이 데이터가 이미 위상차 계산이 되어있다고 가정하고 프로젝트를 진행하였다.

-------------------

1. [01_load_and_save_minmax_and_plot](01_load_and_save_minmax_and_plot.ipynb) : 주어진 csv파일에서 데이터 분석, 필요한 feature 저장, image파일로 변환 및 저장(CNN 활용을 위해서)
2. [02_draw_plot_by_dic_minmax.ipynb](02_draw_plot_by_dic_minmax.ipynb) : 데이터 시각화 및 linear Regression 수행
3. [03_apply_dbscan.ipynb](03_apply_dbscan.ipynb) : DBSCAN로 clustering 수행
4. [04_soft_clustering.ipynb](04_soft_clustering.ipynb) : k-means++를 통해 r+s+t의 합한 값의 max, min값을 clustering한 결과. 각 클러스터에 속할 확률 구하기

## 분류 방법 (4가지)

### 1. [DBSCAN ( clustering )을 적용하여 전류계 고장 여부 확인](03_apply_dbscan.ipynb)
- dbscan_fail_list.pickle에 고장 여부 저장
- DBSCAN은 밀도 기반 데이터 클러스터링 알고리즘으로 복잡한 형상의 데이터셋에도 적용 가능하다. ( 이 데이터셋에 적용하기 가장 적당하다고 생각하였다. )
- '고장난 전류계는 주어진 csv파일에 이상치가 존재할 것이다.'
- 하이퍼파라미터 조정

### 2. Linear Regression으로 고장 여부 확인 -> min, max값을 통해 확인
[소스 보기](02_draw_plot_by_dic_minmax.ipynb)

- label이 없기 때문에 z값을 구하는 것으로 진행하였다.
- 선형 회귀 적용
- 구한 기울기와 실제 값의 차이를 계산하여 매우 멀다면 이상치로 생각한다.



### 3. [Clustering에서 중심점으로부터 얼마나 먼지 확인하여 정답일 확률 구하기](04_soft_clustering.ipynb)
- probability_kmeans_plus.pickle에 k-means++로 구한 중심점을 통해 각 클러스터에 속할 확률 저장
- 각 파일의 **r+s+t는 중성선**으로, 전류가 흐르지 않아야 정상이다.(**0에 근사**해야한다.)
- r+s+t의 max값과 min값이 0과 근사하다면 정상, 차이가 크게난다면 비정상으로 가정한다.
- 각 중심점과 떨어진 거리를 계산하여 정상/비정상 클러스터에 속할 확률을 구한다.

#### 사용 기술
- FCM ( Fuzzy C Means )
- K-means++


### 4. 
