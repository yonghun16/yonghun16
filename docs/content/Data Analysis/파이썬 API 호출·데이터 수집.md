## 학습 목표와 전체 흐름

목표는 파이썬으로 공공데이터 API를 호출하고, 응답 JSON을 확인해 데이터를 수집하는 것이다. 실습 소재는 서울 가락도매시장의 2015\~2024년 배추 가격이며, 품종은 봄·여름(고랭지)·가을·월동 4가지다.

수집은 아래 순서로 진행한다.

1. 요청 URL과 조건(`params`)을 만든다.
2. `requests.get`으로 호출한다.
3. 상태코드와 결과코드로 호출 성공을 확인한다.
4. `response.json()`으로 JSON을 딕셔너리로 바꾼다.
5. `["response"]["body"]`에서 `totalCount`와 `items`를 꺼낸다.
6. 연도와 품종을 바꿔 가며 반복 수집한다.
7. 다음 스텝에서 pandas로 전처리와 시각화를 한다.

## API 호출 기본

API 호출은 `requests.get(URL, params=딕셔너리)` 한 줄이 핵심이며, 조건은 URL에 직접 이어 붙이지 않고 `params` 딕셔너리에 담는다. `requests`가 `?키=값&키=값` 형태의 쿼리 스트링으로 알아서 바꿔 준다.

문제에 나온 요청 URL의 각 조각은 아래 `params` 항목과 1:1로 대응한다.

| URL 쿼리 | params 키 | 의미 |
| --- | --- | --- |
| `serviceKey` | `serviceKey` | 본인 인증키 |
| `pageNo`, `numOfRows` | `pageNo`, `numOfRows` | 페이지 번호, 한 페이지 건수(200) |
| `returnType=JSON` | `returnType` | 응답 형식 |
| `cond[exmn_ymd::GTE]`, `LTE` | 같은 이름 | 조사일자 시작·끝 (예: 20150101\~20151231) |
| `cond[vrty_cd::EQ]` | 같은 이름 | 품종코드 (01 봄, 02 여름, 03 가을, 06 월동) |
| `cond[mrkt_cd::EQ]` 등 | 같은 이름 | 시장·품목·등급 등 고정 조건 |

```python
import requests

URL = "https://apis.data.go.kr/B552845/perDay/price"

params = {
    "serviceKey": SERVICE_KEY,
    "pageNo": 1,
    "numOfRows": 200,
    "returnType": "JSON",
    "cond[exmn_ymd::GTE]": "20150101",
    "cond[exmn_ymd::LTE]": "20151231",
    "cond[se_cd::EQ]": "02",
    "cond[ctgry_cd::EQ]": "200",
    "cond[item_cd::EQ]": "211",
    "cond[grd_cd::EQ]": "04",
    "cond[sgg_cd::EQ]": "1101",
    "cond[mrkt_cd::EQ]": "0110211",
    "cond[vrty_cd::EQ]": "01",
}

response = requests.get(URL, params=params, timeout=30)
```

`timeout=30`은 서버가 응답하지 않을 때 무한 대기하지 않게 하는 안전장치다.

## 응답 확인과 JSON 구조 파악

호출이 성공했는지는 세 단계로 확인한다. HTTP 상태코드 200, 응답 헤더의 `resultCode` 00, 그리고 `totalCount`가 기대한 건수와 같은지다. 이 실습에서 2015년 봄배추의 정상 건수는 66건이다.

```python
print("HTTP 상태코드:", response.status_code)       # 200이면 통신 성공
body = response.json()["response"]["body"]
print("totalCount:", body["totalCount"])           # 66이면 정상
```

처음 보는 API의 JSON 구조는 `print(response)`로는 알 수 없다. 이 출력은 `<Response [200]>`처럼 응답 객체만 보여 주기 때문이다. 아래 방법으로 내용을 직접 열어 봐야 한다.

| 상황 | 방법 |
| --- | --- |
| 전체 구조를 한눈에 보고 싶다 | `print(json.dumps(data, ensure_ascii=False, indent=2))` |
| 데이터가 커서 너무 길다 | `data.keys()`로 한 겹씩 내려가며 확인 |
| `.json()`에서 에러가 난다 | `print(response.text[:500])`로 원본 텍스트 확인 |

한 겹씩 내려가는 방식은 아래처럼 쓴다.

```python
data = response.json()
print(data.keys())                          # ['response']
print(data["response"].keys())              # ['header', 'body']
print(data["response"]["body"].keys())      # ['items', 'totalCount', 'pageNo', 'numOfRows']
```

공공데이터포털 활용신청 페이지의 응답 메시지 예시나 Swagger 미리보기로도 코드를 쓰기 전에 구조를 볼 수 있다.

## JSON 파싱과 딕셔너리 접근

`body = response.json()["response"]["body"]` 한 줄은 JSON 문자열을 파이썬 딕셔너리로 바꾼 뒤, 필요한 `body` 부분만 꺼내 변수에 담는다. Java의 `Map`으로 JSON을 파싱한 뒤 키를 두 번 따라 들어가는 것과 같다.

응답은 아래와 같은 중첩 구조다.

```json
{
  "response": {
    "header": { "resultCode": "00", "resultMsg": "NORMAL SERVICE." },
    "body": {
      "items": { "item": [ {...}, {...} ] },
      "totalCount": 66,
      "pageNo": 1,
      "numOfRows": 200
    }
  }
}
```

- `response.json()`은 JSON 문자열을 딕셔너리로 변환한다.
- `["response"]`와 `["body"]`는 키를 한 겹씩 따라 들어간다.
- 데이터 리스트는 `body["items"]["item"]`에 있다. `body["items"]`만 쓰면 `{"item": [...]}`가 통째로 나온다.
- 키 이름이 틀리거나 인증키 오류로 에러 응답이 오면 `KeyError`가 난다. 이때는 `print(response.json())`으로 실제 응답을 먼저 본다.

같은 동작을 풀어 쓰면 아래와 같다.

```python
data = response.json()          # JSON → dict
body = data["response"]["body"]
items = body["items"]["item"]   # 데이터 리스트
print(items[0])                 # 첫 번째 데이터 1건
```

## 출력 다듬기: indent와 슬라이싱

`print(body["items"])`는 파이썬 딕셔너리를 한 줄로 길게 찍어서 읽기 어렵다. `json.dumps`에 `indent`를 주면 JSON처럼 들여쓰기된 구조로 볼 수 있다.

```python
import json

print(json.dumps(body["items"]["item"][:3], ensure_ascii=False, indent=2))
```

| 요소 | 역할 |
| --- | --- |
| `indent=2` | 2칸 들여쓰기로 구조를 보여 준다 |
| `ensure_ascii=False` | 한글이 `\ucc44\uc18c\ub958` 같은 이스케이프로 나오는 것을 막는다 |
| `[:3]` | 리스트 앞 3건만 잘라 출력한다 (슬라이싱) |

66건을 전부 들여쓰기로 출력하면 화면이 너무 길어 캡처 한 장에 담기지 않는다. 호출 성공 확인이 목적이면 `totalCount`와 앞 2\~3건이면 충분하다.

## 페이지네이션과 반복 수집

API는 한 번에 `numOfRows`건까지만 돌려주므로, 전체 건수(`totalCount`)가 더 많으면 `pageNo`를 올려 가며 여러 번 호출해야 한다. 이 실습은 연도별·품종별로 건수가 200건 이하라 대부분 1페이지로 끝나지만, 안전하게 페이지 루프를 둔다.

```python
def fetch_data(year, code):
    rows, page = [], 1
    while True:
        body = get_page(year, code, page)
        items = body.get("items", {}).get("item", [])
        rows.extend(items)
        if not items or page * 200 >= int(body["totalCount"]):
            break          # 더 가져올 데이터가 없으면 종료
        page += 1
    return rows
```

종료 조건은 두 가지다. 받은 항목이 비었거나, 지금까지 요청한 건수(`page * 200`)가 `totalCount` 이상이면 멈춘다.

연도 10개와 품종 4개를 모두 돌리면 호출은 최소 40번이다. 아래처럼 이중 반복으로 전체를 모은다.

```python
data = []
for year in range(2015, 2025):
    for code in ["01", "02", "03", "06"]:
        data.extend(fetch_data(year, code))
```

단건 응답일 때 `item`이 리스트가 아니라 딕셔너리 하나로 오는 경우가 있다. 그래서 실제 코드에서는 `isinstance(items, dict)`로 확인해 리스트로 감싸 주는 방어 코드를 넣었다.

## 파이썬다운 문법

C/Java식 중첩 반복문과 임시 리스트는 파이썬에서 컴프리헨션과 제너레이터 표현식으로 줄여 쓰는 경우가 많다. 아래 두 코드는 결과가 같다.

```python
# C/Java식
data = []
for year in YEARS:
    for code in VARIETIES:
        for row in fetch_data(year, code):
            data.append(row)

# 파이썬식: 리스트 컴프리헨션 (이중·삼중 for를 한 줄로)
data = [row for year in YEARS for code in VARIETIES for row in fetch_data(year, code)]
```

| 문법 | 예시 | 설명 |
| --- | --- | --- |
| 리스트 컴프리헨션 | `[x for x in a]` | 리스트를 만든다. for 순서가 바깥 → 안쪽 반복문 순서와 같다 |
| 제너레이터 표현식 | `(x for x in a)` | 괄호를 쓰면 리스트를 미리 만들지 않고 필요할 때 하나씩 꺼낸다 |
| 딕셔너리 `.get` | `body.get("items", {})` | 키가 없어도 에러 대신 기본값을 돌려준다 |
| 조건부 표현식 | `a if cond else b` | Java의 삼항 연산자 `cond ? a : b`에 해당한다 |
| `isinstance` | `isinstance(items, dict)` | 값의 자료형을 확인한다 |

컴프리헨션은 한 줄이 길어지면 오히려 읽기 어렵다. for가 세 겹을 넘거나 조건이 복잡해지면 일반 반복문으로 풀어 쓰는 편이 낫다.

## pandas 활용 (다음 스텝)

수집이 끝난 JSON 리스트는 `pd.DataFrame`에 바로 넣어 표로 바꿀 수 있고, 전처리와 분석은 이 표 위에서 한다. 이번 과제(문 1-2)는 수집까지만 요구하므로, 아래 내용은 다음 문제를 위한 준비로 정리해 둔다.

```python
import pandas as pd

df = pd.DataFrame(data)    # JSON 리스트(딕셔너리 목록) → 표
```

| 작업 | 코드 |
| --- | --- |
| 여러 표 합치기 | `pd.concat(frames, ignore_index=True)` |
| 날짜 변환 | `pd.to_datetime(df["exmn_ymd"], format="%Y%m%d")` |
| 가격을 숫자로 변환 | `pd.to_numeric(df["exmn_dd_prc"])` |
| 정렬 | `df.sort_values(["vrty_cd", "exmn_ymd"])` |
| 품종별 건수 | `df.groupby("vrty_cd").size()` |
| 연도·품종별 평균 가격 | `df.groupby([df["exmn_ymd"].dt.year, "vrty_nm"])["exmn_dd_prc"].mean().unstack()` |
| CSV 저장 | `df.to_csv("cabbage_prices.csv", index=False, encoding="utf-8-sig")` |

- API는 날짜와 가격까지 전부 문자열로 주기 때문에, 분석 전에 `to_datetime`과 `to_numeric` 변환이 필요하다.
- `encoding="utf-8-sig"`로 저장해야 엑셀에서 한글이 깨지지 않는다.
- 한 번 CSV로 저장해 두면 다음부터는 API를 40번 다시 호출하지 않고 `pd.read_csv`로 불러 쓸 수 있다.

## 보안: API 키 관리

API 인증키는 비밀번호처럼 다뤄야 하며, 코드에 직접 적은 채로 제출하거나 GitHub에 올리면 다른 사람이 그대로 쓸 수 있다. 이미 노출됐다면 공공데이터포털에서 키를 재발급받는 것이 안전하다.

환경변수로 읽으면 코드에 키가 남지 않는다.

```python
import os

SERVICE_KEY = os.environ["DATA_GO_KR_KEY"]
```

- 터미널에서 먼저 `export DATA_GO_KR_KEY="발급받은_키"`로 설정한 뒤 실행한다.
- 과제 제출용 코드와 캡처에는 키를 `"발급받은_키"`로 바꾸거나 일부를 `****`로 가린다.
- 키가 든 파일은 `.gitignore`에 넣어 저장소에 올라가지 않게 한다.

## 과제 문 1-2 최종 코드

문 1-2는 API 호출 성공과 JSON 수신을 확인하는 것까지만 요구하므로, 코드는 호출 한 번과 확인 출력으로 끝낸다. 아래는 2015년 봄배추를 조회해 66건이 나오는지 확인하는 최소 코드다.

```python
import json
import requests

SERVICE_KEY = "발급받은_키"
URL = "https://apis.data.go.kr/B552845/perDay/price"

params = {
    "serviceKey": SERVICE_KEY,
    "pageNo": 1,
    "numOfRows": 200,
    "returnType": "JSON",
    "cond[exmn_ymd::GTE]": "20150101",
    "cond[exmn_ymd::LTE]": "20151231",
    "cond[se_cd::EQ]": "02",
    "cond[ctgry_cd::EQ]": "200",
    "cond[item_cd::EQ]": "211",
    "cond[grd_cd::EQ]": "04",
    "cond[sgg_cd::EQ]": "1101",
    "cond[mrkt_cd::EQ]": "0110211",
    "cond[vrty_cd::EQ]": "01",
}

response = requests.get(URL, params=params, timeout=30)
print("HTTP 상태코드:", response.status_code)

body = response.json()["response"]["body"]
print("totalCount:", body["totalCount"], "(정상: 66건)")
print(json.dumps(body["items"]["item"][:3], ensure_ascii=False, indent=2))
```

제출 전 캡처 체크리스트는 아래와 같다.

- [ ] 파이썬 코드 전체가 보이는 캡처 (서비스 키는 가림)
- [ ] `HTTP 상태코드: 200`이 출력된 실행 결과
- [ ] `totalCount: 66`이 출력된 실행 결과
- [ ] 들여쓰기된 JSON 데이터가 보이는 실행 결과
- [ ] 문제에서 요구한 품종(봄·여름(고랭지)·가을·월동) 4개를 모두 수집했는지 확인 (과제에서 4개 품종 수집 증거가 필요하면 `vrty_cd`만 바꿔 같은 코드를 반복 호출)

## 자주 만나는 에러와 해결법

API 수집에서 막히는 지점은 대부분 인증, 응답 형식, 구조 접근 세 가지다. 에러가 나면 먼저 `response.status_code`와 `response.text[:500]`으로 서버가 실제로 무엇을 보냈는지 확인한다.

| 증상 | 원인 | 해결 |
| --- | --- | --- |
| `KeyError: 'response'` | 인증키 오류 등으로 정상 JSON 대신 에러 응답이 옴 | `print(response.text[:500])`으로 원본 확인, 키와 활용신청 승인 여부 점검 |
| `JSONDecodeError` | 응답이 JSON이 아니라 XML이나 에러 문구 | `returnType=JSON` 확인, 원본 텍스트 출력 |
| `requests.exceptions.HTTPError` | `raise_for_status()`가 4xx·5xx 상태코드를 감지 | 상태코드와 요청 URL 확인, 잠시 뒤 재시도 |
| `Timeout` | 서버 응답 지연 | `timeout` 값을 늘리거나 재시도 |
| `totalCount`가 0 또는 기대와 다름 | 조건 코드가 틀렸거나 해당 기간 데이터가 없음 | `cond[...]` 값과 날짜 범위 점검 |
| 단건인데 반복문에서 오류 | `item`이 리스트가 아니라 딕셔너리 하나로 옴 | `isinstance(items, dict)`로 확인해 리스트로 감싼다 |
| 인증키 로그인 오류 | 일반 인증키(Encoding/Decoding) 혼동 | 포털에서 받은 키를 그대로 `serviceKey`에 넣어 보고, 안 되면 다른 쪽 키로 교체 |

디버깅의 기본 순서는 상태코드 → 원본 텍스트 → JSON 구조 → 키 접근이다. 위에서부터 차례로 확인하면 어느 단계에서 막혔는지 바로 알 수 있다.
