# 📌 빌딩 조망권 문제 – 알고리즘 TIL

## 0️⃣ 문제 유형 정리

### ✅ 유형
- **슬라이딩 윈도우(Sliding Window)**
- **국소 범위 비교(Local range comparison)**
- (부가적으로) **범위 내 최대값 계산(Range max)**

### ✅ 이 유형의 핵심 설명
이 유형은 “전체를 다 보지 않고, **기준점 주변의 고정된 범위만 보면서 판단**”하는 문제다.  
윈도우(관찰 범위)가 한 칸씩 이동하고, 매 위치에서 **:  
- 최대/최소/합/평균 같은 값을 구하거나  
- 조건을 만족하는지 검사한다.

이 문제에서는 윈도우 크기가 고정(좌우 2칸)이고,  
매 idx마다 주변 4개 값의 **최대 높이**만 알면 조망권이 결정된다.

### ✅ 같은 유형의 대표 문제들
- **연속된 k개 구간의 합/평균/최대/최소** (예: “길이 k 부분합 최댓값”)
- **특정 범위 내 조건 만족 여부** (예: “주변 k칸 안에 더 큰 값이 있는가?”)
- **문자열에서 길이 k 윈도우로 조건 검사** (예: “길이 k인 부분 문자열 중 조건 만족 개수”)
- **두 포인터/윈도우 이동 기반 문제**  
  - 윈도우 길이가 고정이면 Sliding Window  
  - 윈도우 길이가 변하면 Two Pointers로 자주 이어짐

---

## 1️⃣ 핵심 아이디어
이 문제의 핵심은 **각 빌딩을 기준으로 좌우 2칸(총 4개 이웃)만 비교**하면 조망권을 판단할 수 있다는 점이다.  
즉, 전체 빌딩을 다 비교하는 문제가 아니라 **국소 범위(Local range) 비교**로 해결된다.

조망권이 있는 경우, 해당 빌딩의 조망권 수는 다음과 같이 계산된다.

- `현재 빌딩 높이 - (주변 4개 빌딩 중 최대 높이)`

---

## 2️⃣ 해결 전략
1. 양쪽 끝 2칸은 조망권 계산 대상이 아니므로 제외한다.
2. 각 빌딩(idx)에 대해 주변 4개 빌딩(idx-2, idx-1, idx+1, idx+2)을 확인한다.
3. 주변에 **나보다 높거나 같은 빌딩이 하나라도 존재하면** 조망권은 없다.
4. 주변에 **나보다 큰 빌딩이 없다면**, 주변 빌딩들 중 **최대 높이**를 구한다.
5. `현재 높이 - 주변 최대 높이`만큼 조망권을 확보하므로 이를 누적한다.

---

## 3️⃣ 구현 포인트 (내 코드 기준)

### ✅ 포인트 1: 주변에 나보다 큰(또는 같은) 빌딩이 있으면 즉시 조망권 없음 처리
내 코드는 주변 4칸을 확인하면서, 하나라도 현재 빌딩 이상이면 바로 `blocked = True`로 처리하고 탈출한다.

```python
blocked = False
for neighbor_idx in range(idx - 2, idx + 3):
    if idx == neighbor_idx:
        continue

    if building_list[neighbor_idx] >= building_list[idx]:
        blocked = True
        break
```

### ✅ 포인트2: 조망권이 가능한 경우를 대비해 주변 최대 높이 갱신
조망권이 막히지 않는 상황에서는, 주변 빌딩들 중 가장 큰 값을 기록해 둔다.
(조망권 계산에서 빼야 하는 기준값이기 때문)

```python
max_neighbor_height = 0

for neighbor_idx in range(idx - 2, idx + 3):
    if idx == neighbor_idx:
        continue

    if building_list[neighbor_idx] >= building_list[idx]:
        blocked = True
        break

    if building_list[neighbor_idx] > max_neighbor_height:
        max_neighbor_height = building_list[neighbor_idx]
```

### ✅ 포인트 3: `blocked` 플래그로 조망권이 있는 경우에만 누적
`blocked == False`인 경우에만 조망권을 더한다.

```python
if blocked == False:
    total_view += building_list[idx] - max_neighbor_height
```

## 4️⃣ 개선 및 확장 아이디어 (코드 포함)
### 🔹 개선 1: 순회 범위를 처음부터 제한해서 if 분기 제거
현재는 전체 인덱스를 돌면서 내부에서 조건으로 걸러낸다.
```python
for idx in range(len(building_list)):
    if 2 <= idx <= len(building_list) - 3:
        ...
```
이 부분은 애초에 바깥 반복문을 유효 범위로 제한하면 더 깔끔해진다.
```python
for idx in range(2, len(building_list) - 2):
    ...
```
- 조건 분기 제거
- 인덱스 범위가 코드 자체에 드러나서 의도가 명확해짐

### 🔹 개선 2: 플래그(blocked) 대신 “차이값(diff)”으로 누적 로직 단순화
현재는 blocked를 통해 조망권 여부를 관리한다.\
대신, 주변 최대 높이를 구한 뒤 차이를 계산해서
양수일 때만 누적하면 조망권 판단이 더 직관적이다.
```python
max_neighbor_height = max(
    building_list[idx - 2],
    building_list[idx - 1],
    building_list[idx + 1],
    building_list[idx + 2]
)

diff = building_list[idx] - max_neighbor_height
if diff > 0:
    total_view += diff
```
- `blocked` 변수가 사라져서 로직이 짧아짐
- “조망권 = 양수 차이만큼”이 코드로 직접 표현됨
- `break` 흐름이 없어져 읽기가 쉬워짐

### 🔹 개선 3: 주변 최대값을 max()로 계산해 갱신 루프 단순화
내 코드에서는 루프를 돌며 max_neighbor_height를 갱신했다.
하지만 주변이 항상 4개로 고정이라면 max()가 더 단순하다.
```python
max_neighbor_height = max(
    building_list[idx - 2],
    building_list[idx - 1],
    building_list[idx + 1],
    building_list[idx + 2]
)
```
- 코드 길이 감소
- 실수(인덱스 누락 등) 가능성 감소