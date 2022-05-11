---
title: "천문학개론2 - 태양 자전주기 구하기"
excerpt: "태양 표면의 활동지역을 이용하여 태양의 자전 주기를 구해보자."

categories:
  - Astronomy

last_modified_at: 2022-05-10

toc: true
toc_sticky: true
toc_label: "Contents"
---

> 천문학개론2 강의에서 진행했던 실습을 개발자의 눈으로 복습해보자!  
> 원래의 실습에서는 태양의 자전 주기를 구하기 위한 계산식에 중점을 두지만, 이번 복습에서는 파이썬 코드를 통해 데이터를 읽고 처리하는 것에 중점을 두고자 한다.

## 1. 개요

**Helioviewer**에서 **PROBA-2 위성**의 태양 관측 데이터를 파이썬 코드를 통해 처리한다.
처리한 데이터로 태양 표면 활동지역의 이동 속도를 구하고, 이를 이용하여 태양의 자전 주기를 구한다.

## 2. Modules

```python
import numpy as np
import astropy.io.fits as fits
import matplotlib.pyplot as plt
```

- **numpy**: 배열을 생성하고, 배열의 연산을 수행하는 모듈
- **astropy**: 천문학 데이터를 읽고 처리하는 모듈
  - **fits**: FITS 파일을 읽고 처리하는 모듈
- **matplotlib**: 데이터를 시각화해주는 모듈
  - **pyplot**: 그래프를 그리는 모듈

## 3. 데이터 읽기

```python
image_data = fits.open('데이터 주소/데이터 이름.fits')
data = image_data[0].data
header = image_data[0].header
```

앞에서 import한 astropy.io.fits 모듈을 사용하여 FITS 파일을 읽는다. FITS 파일은 header와 data로 구성되어 있다. header와 data를 각각의 변수에 저장한다.

_image_data.info()_ 코드를 실행하면 image_data의 정보가 출력된다.

## 4. 데이터 시각화

```python
# 이 코드는 다음 에러를 무시하기 위한 코드이다.
# RuntimeWarning: divide by zero encountered in log
np.seterr(divide='ignore')

image = np.log(np.array(data))
# 밝기 조절
max_value = np.percentile(image, 100)
min_value = np.percentile(image, 30)

plt.figure(figsize=(8, 8))
plt.imshow(image, cmap='hot', origin='lower',
vmax=max_value, vmin=min_value)
plt.show()
```

**np.array(data)**를 사용하여 데이터를 배열로 변환한다. 이후 **np.log**를 사용하여 데이터를 로그로 변환한다. 천문학 데이터들은 스케일이 크기 때문에 로그 스케일로 변환하는게 좋다!
이제 numpy를 통해 필요한 데이터를 모두 처리했다.

본격적으로 matplotlib을 사용하여 데이터를 시각화해보자. **plt.figure**는 그래프 평면을 생성해준다. **figsize**로 그래프의 크기를 설정하자. **plt.imshow**는 그래프 평면 위에 데이터를 그리는 함수이다. 우리가 시각화 하고싶은 데이터인 **image**를 파라미터로 입력하자. 이후 데이터가 더 잘 보일 수 있게 하기 위해 **cmap**으로 색상을 결정한다. 그리고 **vmax**와 **vmin**을 통해 밝기의 최대값과 최솟값을 설정한다.

여기서 끝난다면 이미지 데이터는 상하반전이 된 채로 보일 것이다! 그 이유는 파이썬 배열의 특성에 있다. 파이썬 배열은 행렬 구조로 행은 위에서 아래로 진행하기 때문에 상하반전이 되는 것이다. 이를 해결하기 위해 **origin='lower'**을 통해 우리가 아는 수학적, 물리적 좌표계로 전환해준다.

이제 **plt.show**를 실행하여 이미지를 console에 출력해보자!
![](/assets/images/astro_solar.png){: .align-center}
다음과 같은 그래프가 출력된다면 성공이다!

## 5. 데이터 활용

이제 관측 시간이 다른 2개 이상의 데이터를 사용하여 그래프를 출력한 뒤, 태양 표면 활동지역의 이동속도를 구하면 태양의 자전주기를 구할 수 있다!

> 태양 자전주기 = 2π × (태양 R) / (태양 표면 활동지역의 ⱱ)
