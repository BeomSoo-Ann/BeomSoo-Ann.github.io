---
title: "천문학개론2 - 보이저 1호 데이터 처리하기"
excerpt: "보이저 1호가 지구로 보낸 데이터를 처리하여 태양권계면을 통과한 시점을 추측해보자"

categories:
  - Astronomy

last_modified_at: 2022-05-17

toc: true
toc_sticky: true
toc_label: "Contents"
---

> 조금 더 복잡한 데이터를 파이썬 코드를 통해 처리해보자!
> 저번 실습에서는 FITS 파일의 데이터를 이미지화했다면, 이번에는 txt 파일의 데이터를 **그래프화** 해보자.

## 1. 개요

[**NASA GSFC**](https://cdaweb.gsfc.nasa.gov)(Goddard Space Flight Center)에서 **Voyager 1**이 전송한 Distance, Magnetic field, Proton Flux data를 받아 파이썬 코드를 통해 그래프로 출력해보자.
이를 통해 Voyager 1이 태양권계면을 통과한 시점을 추측해보자.

## 2. Modules

```python
import datetime
import numpy as np
import matplotlib.pyplot as plt
```

- **datetime**: 날짜, 시간 자료를 처리해주는 모듈
- **numpy**: 배열을 생성하고, 배열의 연산을 수행하는 모듈
- **matplotlib**: 데이터를 시각화해주는 모듈
  - **pyplot**: 그래프를 그리는 모듈

## 3. 데이터 처리

```python
file = '데이터 주소/데이터 이름.txt'
full_text = open(file, 'r')
```

위의 코드를 통해 텍스트 파일을 읽을 것이다. FITS와 다르게 txt는 여러 분야에서 흔히 사용하기 때문에 python 내장 함수인 open으로도 충분히 읽어올 수 있다.

데이터의 구조는 다음과 같다.

![](/assets/images/astro_day2.png){: .align center}
이 파일에서 우리는 date, time, distance, magnetic, proton data를 분리할 것이다.

이 데이터들을 분리하려면 먼저 데이터를 제외한 정보들을 제거해야한다. 이후 각 행을 한 줄씩 읽으면서 각 열의 데이터를 각각 다른 array로 저장하면 된다.
읽어야 할 데이터의 양이 많으니 while문으로 무한 루프를 만든 후 특정 조건에 루프를 벗어나도록 하자.

```python
# 각 데이터를 담을 list를 선언해준다.
date = []
time = []
distance = []
magnetic = []
proton = []

n = 0   # 데이터의 행을 구분해주기 위한 변수 n

while True:
  line = full_text.readline()   # 행을 한 줄씩 불러온다.

  if not line:   # 불러올 행이 없는 경우 루프를 break한다.
    break
  if '#' in line:   # '#'이 들어간 행은 데이터가 없는 행이므로 continue한다.
    continue

  n += 1
  columns = line.split()   # 각 행의 데이터는 space로 구분되어있기 때문에 split해준다.

  if n == 1:   # 첫 줄은 데이터의 정보가 담겨있다. header로 분류해준다.
    header = columns
    continue
  if n == 2:   # 둘째 줄은 데이터의 단위가 담겨있다. units로 분류해준다.
    units = columns
    continue
  # 데이터들 중 크기가 너무 작은 데이터가 존재하는 경우가 있다.
  # 이는 관측기기가 관측을 하지 않았거나 하지 못한 경우이므로 제외해주자.
  if float(columns[3]) <= -1e30 and float(columns[4]) <= -1e30:
    continue

  # 예외사항을 모두 제거하고 남은 데이터들을 각 list에 저장해주자.
  date.append(columns[0])
  time.append(columns[1])
  distance.append(columns[2])
  magnetic.append(columns[3])
  proton.append(columns[4])

# 분류한 데이터들을 처리하기 쉽게 array로 바꿔준다.
distance = np.asarray(distance, dtype=float)
magnetic = np.asarray(magnetic, dtype=float)
proton = np.asarray(proton, dtype=float)
```

> 원래 실습 코드는 if문 아래에 또 if문 넣고, 그 밑에 또 if문 들어가고 자꾸 반복되길래 예외 조건만 꺼내서 continue하게 만들어 깔끔하게 정리해봤다.
> 글로 설명하기에는 너무 장황해져서 각 코드에 주석을 달아 설명을 줄이도록 하겠다.

- **array()와 asarray()의 차이점**  
  array(): 데이터의 중복 여부와 상관없이 array를 만들어준다.  
  asarray(): 데이터가 다를 경우에만 copy를 한다.

날짜 데이터는 아직 date와 time 두 개의 데이터로 나누어져있다. 이 둘을 합쳐 하나의 데이터로 만들어보자.

```python
# 두 데이터를 병합하여 하나의 list에 담을 것이다.
dates = []
# date 구조: 00-00-0000
# time 구조: 00:00:00.000
for i in range(0, len(date)):
    date1 = date[i].split('-')
    time1 = time[i].split(':')
    sec = time1[2].split('.')

    day = int(date1[0])
    month = int(date1[1])
    year = int(date1[2])

    hour = int(time1[0])
    minute = int(time1[1])
    second = int(sec[0])
    msec = int(sec[1])

    one_day = datetime.datetime(year, month, day, hour, minute, second, msec)
    dates.append(one_day)
```

> 원래 코드는 데이터 순서를 일일이 세어서 day, month, year 등을 하나씩 나누었다.
> 그런데 이러한 작업을 굳이 할 필요가 있을까 싶어 split()을 사용하는 코드로 바꾸어보았다.

## 4. 데이터 시각화

데이터를 모두 분류했으니, 분류한 데이터로 그래프를 그려보자!

```python
x_data = distance # distance or dates
y_data = proton # proton or magentic / x_data가 dates일 경우 distance

plt.figure(figsize=(16, 8))
plt.plot(x_data, y_data)
# dates 이용시
# plt.plot_date(dates, y_data)
plt.show()
```

![](/assets/images/day2/Figure_1.png){: .align-center}

x축 data를 distance, y축 data를 proton으로 설정한 그래프이다. 그래프는 잘 나왔지만, 생각보다 너무 밋밋하다. 그래프에 여러 가지 설명을 추가해보자!

```python
plt.title("Distance-Proton", fontsize=15)   # 그래프의 Title
plt.xlabel("Distance" + " ("+units[2]+")", fontsize=15)  # x축 label
plt.ylabel("Proton"+" ("+units[4]+")", fontsize=15)   # y축 label
plt.xticks(fontsize=15) # x축 숫자 fontsize
plt.yticks(fontsize=15) # y축 숫자 fontsize
plt.legend(["Proton"], fontsize=15) # 그래프 우측 상단에 data name

# data가 급격히 변화하는 시점을 선으로 그어 뚜렷하게 나타내보자.
# 변화 시점을 보니 Distance가 121.6AU일 때가 가장 급격히 변화한 것 같다
plt.axvline(121.6, color="r")
```

![](/assets/images/day2/Figure_2.png){: .align-center}

여러 가지 설명들이 추가되었다!

위 그래프를 보면 Voyager 1은 태양으로부터 **121.6 AU**를 날아갔을 때 태양권계면을 통과했다는 사실을 알 수 있다!

> 코드에서 data값만 바꿔주면 Distance-Magnetic graph, Dates-Distance graph 등의 그래프도 얻을 수 있다.
> 자세한 통과 시점을 얻기 위해 다른 그래프도 출력해보자.
