---
title: "천문학개론2 - 태양 자전주기 구하기"
excerpt: "태양 앞에 있는 금성을 이용하여 태양의 자전 주기를 구해보자."

categories:
  - Astronomy

last_modified_at: 2022-05-10

toc: true
toc_sticky: true
---

> 천문학개론2 강의에서 진행했던 실습을 개발자의 눈으로 복습해보자!  
> 원래의 실습에서는 태양의 자전 주기를 구하기 위한 계산식에 중점을 두지만, 이번 복습에서는 파이썬 코드를 통해 데이터를 읽고 처리하는 것에 중점을 두고자 한다.

## 1. 개요

**Helioviewer**에서 **PROBA-2 위성**의 태양 관측 데이터를 파이썬 코드를 통해 처리한다.
처리한 데이터로 태양 앞을 지나가는 금성의 이동 속도를 구하고, 금성의 이동 속도를 이용하여 태양의 자전 주기를 구한다.

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
