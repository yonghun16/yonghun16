# 최대공약수(GCD)와 최소공배수(LCM) 완전정복

## 📚 목차
1. [기본 개념](#기본-개념)
2. [유클리드 호제법](#유클리드-호제법)
3. [GCD 구하기](#gcd-구하기)
4. [LCM 구하기](#lcm-구하기)
5. [3개 이상의 수 처리](#3개-이상의-수-처리)
6. [실전 코드](#실전-코드)
7. [자주 하는 실수](#자주-하는-실수)
8. [실무 예제](#실무-예제)

---

## 기본 개념

### 최대공약수(GCD: Greatest Common Divisor)
**정의:** 두 수 이상의 공통된 약수 중 가장 큰 수

```
예시: GCD(12, 18)
- 12의 약수: 1, 2, 3, 4, 6, 12
- 18의 약수: 1, 2, 3, 6, 9, 18
- 공약수: 1, 2, 3, 6
- 최대공약수: 6
```

### 최소공배수(LCM: Least Common Multiple)
**정의:** 두 수 이상의 공통된 배수 중 가장 작은 수

```
예시: LCM(12, 18)
- 12의 배수: 12, 24, 36, 48, 60, ...
- 18의 배수: 18, 36, 54, 72, ...
- 공배수: 36, 72, 108, ...
- 최소공배수: 36
```

### GCD와 LCM의 관계
```
GCD(a, b) × LCM(a, b) = a × b

예: GCD(12, 18) × LCM(12, 18) = 12 × 18
    6 × 36 = 216 ✓
```

---

## 유클리드 호제법

### 원리
두 수의 최대공약수를 효율적으로 구하는 알고리즘

```
GCD(a, b) = GCD(b, a mod b)
a mod b == 0일 때, GCD = b
```

### 단계별 설명

**GCD(12, 18) 계산:**
```
1단계: GCD(12, 18) → 12 % 18 = 12
2단계: GCD(18, 12) → 18 % 12 = 6
3단계: GCD(12, 6) → 12 % 6 = 0
4단계: GCD(6, 0) → 답: 6 ✓
```

### 시간복잡도
- **시간복잡도:** O(log(min(a, b)))
- **매우 효율적!** (대수시간)

---

## GCD 구하기

### 1️⃣ 재귀 함수 (직관적)

```python
def gcd(a: int, b: int) -> int:
    """유클리드 호제법을 사용한 GCD"""
    if a % b == 0:
        return b
    else:
        return gcd(b, a % b)

# 테스트
print(gcd(12, 18))  # 6
print(gcd(48, 18))  # 6
```

**장점:** 호제법의 원리가 명확함  
**단점:** 깊은 재귀가 필요한 경우 스택 오버플로우 가능

### 2️⃣ 반복 함수 (효율적)

```python
def gcd(a: int, b: int) -> int:
    """반복으로 GCD 구하기"""
    while b != 0:
        a, b = b, a % b
    return a

# 테스트
print(gcd(12, 18))  # 6
```

**장점:** 스택 오버플로우 없음, 간결함  
**단점:** 약간의 복잡성

### 3️⃣ math.gcd() 사용 (권장)

```python
import math

result = math.gcd(12, 18)
print(result)  # 6
```

**장점:** 가장 간단, 최적화됨  
**단점:** 내부 동작 원리 이해 필요

---

## LCM 구하기

### 공식
```
LCM(a, b) = (a × b) / GCD(a, b)
```

### 1️⃣ GCD를 이용한 방법

```python
def gcd(a: int, b: int) -> int:
    if a % b == 0:
        return b
    return gcd(b, a % b)

def lcm(a: int, b: int) -> int:
    """GCD를 이용한 LCM"""
    return (a * b) // gcd(a, b)

# 테스트
print(lcm(12, 18))  # 36
print(lcm(12, 18) == 12 * 18 // gcd(12, 18))  # True
```

### 2️⃣ math.lcm() 사용 (Python 3.9+)

```python
import math

result = math.lcm(12, 18)
print(result)  # 36
```

---

## 3개 이상의 수 처리

### 핵심 원리
```
GCD(a, b, c) = GCD(GCD(a, b), c)
LCM(a, b, c) = LCM(LCM(a, b), c)
```

→ **재귀적으로 2개씩 계산**

### 1️⃣ for 루프 사용

```python
import math

def find_gcd_multiple(numbers: list) -> int:
    """여러 수의 GCD"""
    result = numbers[0]
    for num in numbers[1:]:
        result = math.gcd(result, num)
    return result

def find_lcm_multiple(numbers: list) -> int:
    """여러 수의 LCM"""
    def lcm(a, b):
        return (a * b) // math.gcd(a, b)
    
    result = numbers[0]
    for num in numbers[1:]:
        result = lcm(result, num)
    return result

# 테스트
data = [12, 18, 24]
print(find_gcd_multiple(data))  # 6
print(find_lcm_multiple(data))  # 72
```

### 2️⃣ reduce() 사용 (함수형, 권장)

```python
from functools import reduce
import math

def find_gcd_multiple(numbers: list) -> int:
    """reduce를 이용한 GCD"""
    return reduce(math.gcd, numbers)

def find_lcm_multiple(numbers: list) -> int:
    """reduce를 이용한 LCM"""
    lcm_func = lambda a, b: (a * b) // math.gcd(a, b)
    return reduce(lcm_func, numbers)

# 테스트
data = [12, 18, 24]
print(find_gcd_multiple(data))  # 6
print(find_lcm_multiple(data))  # 72
```

### 3️⃣ Python 3.9+ math.gcd/lcm()

```python
import math
from functools import reduce

data = [12, 18, 24]
gcd_result = reduce(math.gcd, data)
lcm_result = reduce(math.lcm, data)

print(gcd_result)  # 6
print(lcm_result)  # 72
```

---

## 실전 코드

### 정올 문제 1002 - 최종 버전

```python
# ============================================
# [JOL] 최대공약수, 최소공배수
# Link: https://jungol.co.kr/problem/1002
# Level: Bronze 3
# ============================================

import os
import sys
from functools import reduce
from typing import List

if os.path.exists("./input_test.txt"):
    sys.stdin = open("./input_test.txt", "r", encoding="utf-8")

input = sys.stdin.readline
sys.setrecursionlimit(10**6)


def get_input_data() -> List[int]:
    """입력 데이터를 받아서 정수 리스트로 반환합니다."""
    n = int(input())
    return list(map(int, input().split()))


def gcd(a: int, b: int) -> int:
    """유클리드 호제법을 사용하여 최대공약수를 구합니다.
    
    Args:
        a: 첫 번째 수
        b: 두 번째 수
        
    Returns:
        a와 b의 최대공약수
    """
    return b if a % b == 0 else gcd(b, a % b)


def lcm(a: int, b: int) -> int:
    """두 수의 최소공배수를 구합니다.
    
    Args:
        a: 첫 번째 수
        b: 두 번째 수
        
    Returns:
        a와 b의 최소공배수
    """
    return (a * b) // gcd(a, b)


def solution(data: List[int]) -> None:
    """여러 수의 최대공약수와 최소공배수를 구해서 출력합니다."""
    result_gcd = reduce(gcd, data)
    result_lcm = reduce(lcm, data)
    
    print(result_gcd, result_lcm)


if __name__ == "__main__":
    solution(get_input_data())
```

### 입출력 예시

```
입력:
3
12 18 24

출력:
6 72

설명:
- GCD(12, 18, 24) = 6
- LCM(12, 18, 24) = 72
```

---

## 자주 하는 실수

### ❌ 실수 1: LCM 계산 시 잘못된 GCD 사용

```python
# 잘못된 코드
for i in range(2, len(data)):
    gcd_val = get_gcd(gcd_val, data[i])  # 전체 GCD
    lcm_val = get_lcm(lcm_val, data[i], gcd_val)  # ❌ 틀림!
```

**문제:** LCM(a, b) = a × b / GCD(a, b)에서  
- `a`는 현재까지의 lcm_val
- `b`는 data[i]
- GCD는 `lcm_val과 data[i]의 GCD`여야 함

```python
# 올바른 코드
for i in range(2, len(data)):
    gcd_val = get_gcd(gcd_val, data[i])
    gcd_lcm_data = get_gcd(lcm_val, data[i])  # ✅ 올바름
    lcm_val = get_lcm(lcm_val, data[i], gcd_lcm_data)
```

### ❌ 실수 2: 빈 리스트 처리 안 함

```python
# 잘못된 코드
def find_gcd_multiple(numbers: list) -> int:
    return reduce(math.gcd, numbers)  # 빈 리스트면 에러!

# 올바른 코드
def find_gcd_multiple(numbers: list) -> int:
    if not numbers:
        return 0  # 또는 예외 처리
    return reduce(math.gcd, numbers)
```

### ❌ 실수 3: 음수 처리 미흡

```python
# 음수가 포함된 경우
import math

print(math.gcd(12, -18))  # 6 (Python은 절댓값으로 처리)
print(math.gcd(-12, -18))  # 6
```

### ❌ 실수 4: 타입 불일치

```python
# 잘못된 코드
result = gcd(12.5, 18)  # float는 안됨!

# 올바른 코드
result = gcd(12, 18)  # 정수만 사용
```

---

## 실무 예제

### 예제 1: 케이크 같은 크기로 자르기

```python
from functools import reduce
import math

def cut_cakes(sizes: list) -> int:
    """케이크들을 같은 크기로 자를 수 있는 최대 크기"""
    return reduce(math.gcd, sizes)

cakes = [12, 18, 24]  # cm
max_piece_size = cut_cakes(cakes)
print(f"최대 자를 수 있는 크기: {max_piece_size}cm")  # 6cm
```

### 예제 2: 신호 동기화

```python
from functools import reduce
import math

def sync_signals(frequencies: list) -> int:
    """여러 신호가 동시에 신호를 보내는 최소 시간"""
    return reduce(lambda a, b: (a * b) // math.gcd(a, b), frequencies)

signals = [4, 6, 8]  # Hz
sync_time = sync_signals(signals)
print(f"동기화 시간: {sync_time}초")  # 24초
```

### 예제 3: 타일로 바닥 채우기

```python
from functools import reduce
import math

def calculate_tile_size(width: int, height: int) -> int:
    """직사각형을 정사각형 타일로 채울 때 최대 타일 크기"""
    return math.gcd(width, height)

room_width, room_height = 12, 18  # m
tile_size = calculate_tile_size(room_width, room_height)
print(f"최대 타일 크기: {tile_size}m")  # 6m
```

### 예제 4: 최대공약수와 최소공배수 동시 계산

```python
from functools import reduce
import math

def gcd_lcm_calculator(numbers: list) -> tuple:
    """여러 수의 GCD와 LCM을 동시에 계산"""
    gcd_val = reduce(math.gcd, numbers)
    
    lcm_func = lambda a, b: (a * b) // math.gcd(a, b)
    lcm_val = reduce(lcm_func, numbers)
    
    return gcd_val, lcm_val

data = [12, 18, 24]
gcd_result, lcm_result = gcd_lcm_calculator(data)
print(f"GCD: {gcd_result}, LCM: {lcm_result}")  # GCD: 6, LCM: 72
```

---

## 📊 성능 비교

| 방법 | 시간복잡도 | 공간복잡도 | 추천 |
|------|----------|----------|------|
| 재귀 호제법 | O(log(min(a,b))) | O(log(min(a,b))) | ⭐⭐⭐ |
| 반복 호제법 | O(log(min(a,b))) | O(1) | ⭐⭐⭐⭐ |
| math.gcd() | O(log(min(a,b))) | O(1) | ⭐⭐⭐⭐⭐ |
| math.lcm() | O(log(min(a,b))) | O(1) | ⭐⭐⭐⭐⭐ |

---

## 💡 핵심 정리

### GCD 구하기
```python
# 가장 간단 (Python 3.9+)
import math
math.gcd(12, 18)  # 6

# 3개 이상
from functools import reduce
reduce(math.gcd, [12, 18, 24])  # 6
```

### LCM 구하기
```python
# 가장 간단 (Python 3.9+)
import math
math.lcm(12, 18)  # 36

# 3개 이상
from functools import reduce
import math
reduce(lambda a, b: a * b // math.gcd(a, b), [12, 18, 24])  # 72
```

### 중요 공식
```
GCD(a, b) × LCM(a, b) = a × b
GCD(a, b, c) = GCD(GCD(a, b), c)
LCM(a, b, c) = LCM(LCM(a, b), c)
```

---

## 🎯 연습 문제

1. GCD(48, 18)을 구하시오.
2. LCM(12, 15)를 구하시오.
3. [24, 36, 48]의 GCD와 LCM을 구하시오.
4. 60cm와 90cm의 두 길이를 정사각형 타일로 정확히 채우려면, 최대 타일 크기는?
5. 시계 A는 4시간마다, B는 6시간마다 울린다. 둘이 동시에 울리는 최소 시간은?

**답:** 
1. 6
2. 60
3. GCD=12, LCM=144
4. 30cm
5. 12시간

---

**학습 완료!** 🎉  
이제 GCD와 LCM의 기본부터 응용까지 완벽하게 이해했습니다!
