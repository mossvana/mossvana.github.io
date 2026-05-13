---
layout: post
title: "2026 핵테온 세종 write-up"
date: 2026-05-13
categories: [ctf]
---
<br>

# Hacktheon 2026 CTF Writeup

이번 포스트에서는 Hacktheon 2026에서 풀었던 세 문제를 정리한다.

---
<br>

## 1. simple-sqli

### 문제 개요

어드민 계정으로 로그인에 성공하면 플래그를 획득하는 웹 문제이다.

### 풀이

소스코드를 살펴보면 로그인 처리 부분에서 사용자 입력을 별도의 검증 없이 SQL 쿼리에 직접 삽입하고 있었다.

![image](/assets/images/ctf/hacktheon2026/1.png)


아이디 값으로 `admin’--` , 비밀번호로 `1234`를 입력했다. 

![image](/assets/images/ctf/hacktheon2026/2.png)


SQL 쿼리가 다음과 같이 변형되어 비밀번호 검증 부분이 주석 처리(`--`)되는 원리다.

```sql
SELECT * FROM users WHERE id='admin'--' AND password='1234'
-- → WHERE id='admin' 만 남아 어드민으로 로그인 성공
```

![image](/assets/images/ctf/hacktheon2026/3.png)

### 플래그

hacktheon2026{d0nt_f0rget_the_s1ng1e_qu0te}

---
<br>

## 2. Recover It!

### 문제 개요

주어진 파일을 역분석을 통해 플래그를 복원하는 리버싱 문제이다.

### 풀이

#### IDA로 코드 분석

IDA로 바이너리를 열어 분석하면 XOR 암호화 루프가 존재하는 것을 확인할 수 있었다.

![image](/assets/images/ctf/hacktheon2026/4.png)

For문을 통해 입력값을 XOR 연산하고, 그 결과를 `cmptable`에 저장된 값과 비교하는 구조였다.

#### cmptable 값 확인

`cmptable`에서 비교에 사용되는 데이터를 확인했다.

![image](/assets/images/ctf/hacktheon2026/5.png)

64바이트 분량의 비교 데이터를 확인하고, 이를 역으로 XOR 복호화하는 코드를 작성했다.

#### 복호화 코드 작성

![image](/assets/images/ctf/hacktheon2026/6.png)

코드를 실행하면 플래그가 출력된다.

![image](/assets/images/ctf/hacktheon2026/7.png)

### 플래그

Hacktheon2026{22c34e819d2800db605d9fdbc9ba9ab71d6b9175d6b3b016c49cd94624f545c3}

---
<br>

## 3. Plottergeist

### 문제 개요

`.pcap` 파일이 주어지며, 패킷 안에 숨겨진 플래그를 찾는 문제이다.

### 풀이

#### 패킷 추출

`tshark`로 UDP 패킷의 페이로드를 추출했다.

```bash
tshark -r plottergeist.pcap -Y udp -T fields -e udp.payload > payload.txt
```

추출된 페이로드 일부는 다음과 같았다.

![image](/assets/images/ctf/hacktheon2026/8.png)

```
c1464d543a4f507c53517c44547c414d7c424d7e
```

#### 포맷 파악

위 값을 hex → ASCII로 변환하면:

```
FMT:OP|SQ|DT|AM|BM~
```

패킷 포맷을 다음과 같이 정의할 수 있었다.

| 필드 | 의미 |
|------|------|
| `OP` | Operation type |
| `SQ` | Sequence number |
| `DT` | Delta time (시간 간격) |
| `AM` | A 모터 이동량 |
| `BM` | B 모터 이동량 |

#### 모션 패킷 분석

실제 모션 패킷 예시:

```
a100080006001900197e
```

이를 분해하면:

```
a1 | 0008 | 0006 | 0019 | 0019 | 7e
 ↓      ↓       ↓       ↓       ↓    ↓
OP     SQ      DT      AM      BM   ~ (delimiter)
```

- `a1`: motion packet 식별자
- `7e` (`~`): 패킷 구분자

#### 시각화

`55~` 패킷이 pen up/down 토글 신호라고 가정하고, 모터 이동량을 좌표로 누적하여 시각화하는 코드를 작성했다.

![image](/assets/images/ctf/hacktheon2026/9.png)

![image](/assets/images/ctf/hacktheon2026/10.png)

모든 이동을 선분으로 연결하면 겹치는 부분이 많아 글자를 식별하기 어려웠다. 대신 **점의 밀도(scatter plot)**로 확인하니 플래그를 읽을 수 있었다.

```python
import matplotlib.pyplot as plt

def to_signed(v):
    if v >= 0x8000:
        return v - 0x10000
    return v

points1 = []
points2 = []

x1 = y1 = 0
x2 = y2 = 0

with open("payload.txt", "r") as f:
    for line in f:
        line = line.strip()

        if not (line.startswith("a1") and line.endswith("7e")):
            continue

        am = to_signed(int(line[10:14], 16))
        bm = to_signed(int(line[14:18], 16))

        # 공식 1
        dx1 = (am + bm) / 2
        dy1 = (am - bm) / 2
        x1 += dx1
        y1 += dy1
        points1.append((x1, y1))

        # 공식 2
        dx2 = (am - bm) / 2
        dy2 = (am + bm) / 2
        x2 += dx2
        y2 += dy2
        points2.append((x2, y2))

# Plot 1
plt.figure(figsize=(12, 12))
plt.scatter(
    [p[0] for p in points1],
    [p[1] for p in points1],
    s=0.2
)
plt.gca().invert_yaxis()
plt.axis("equal")
plt.title("Formula 1")
plt.show()

# Plot 2
plt.figure(figsize=(12, 12))
plt.scatter(
    [p[0] for p in points2],
    [p[1] for p in points2],
    s=0.2
)
plt.gca().invert_yaxis()
plt.axis("equal")
plt.title("Formula 2")
plt.show()
```

![image](/assets/images/ctf/hacktheon2026/11.png)

### 플래그

```
hacktheon2026{the_plotter_reveals_its_secret_through_sound}
```
