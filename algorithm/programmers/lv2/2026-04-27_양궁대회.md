# 프로그래머스 - 양궁대회 (Lv.2)

&nbsp;

## 📌 문제 요약

라이언이 어피치를 **가장 큰 점수 차이**로 이기도록 화살 n발을 배분하는 문제다.

**규칙**
- 과녁은 10점~0점까지 11칸
- 각 점수에서 화살을 더 많이 쏜 쪽이 해당 점수를 가져감 (동점이면 어피치)
- 둘 다 0발이면 아무도 점수 없음
- 라이언은 반드시 n발을 전부 소모해야 함

| 조건 | 반환 |
|---|---|
| 이길 수 있는 경우 | 점수 차이가 최대인 화살 배분 (길이 11 배열) |
| 최대 점수 차가 동일한 경우가 여럿 | 낮은 점수를 더 많이 맞힌 쪽 우선 |
| 이길 수 없는 경우 (지거나 비김) | `[-1]` |

---

&nbsp;

## 🤔 풀이 흐름

1. 10개의 점수판만 진행하므로 완전탐색으로 진행하여도 시간 초과가 안날거라 생각함
2. DFS를 통하여 10점부터 점수를 계산
3. 만약 `라이언이 남은 화살 개수 > n점 과녁판에 어피치가 맞힌 화살 개수` 일 때 → 라이언 점수 + 과녁판 점수, 화살 개수에서 `어피치가 맞힌 화살 개수 + 1` 차감
4. 라이언이 점수 안가져갈 때 → 화살 낭비 X → 어피치 점수
5. 마지막 0점까지 계산한 이후 제일 낮은 점수를 많이 맞춘 점수판을 리턴

---

&nbsp;

## 🚨 풀이할 때 발생한 문제

### 1. 점수 차이값(`chai`) 갱신 안함

라이언이 이긴 경우에서 라이언과 어피치의 점수차가 제일 큰 과녁판을 리턴하고, 만약 2개 이상일 경우에 제일 낮은 점수를 많이 맞춘 점수판을 리턴해야 하는데 **문제를 대충 읽어서 문제 발생.**

`chai = 0`이므로 계속 answer 값이 잘못된 값으로 업데이트가 됨.

### 2. 낮은 점수를 제일 많이 맞춘 경우를 어떻게 계산할지 모름

처음에는 배열을 뒤집어서 String으로 이어붙인 뒤 `Long`으로 비교하려 했으나, 화살 수가 두 자리 이상이면 자릿수가 밀려서 비교가 깨지는 문제가 있었다.

---

&nbsp;

## ✅ 풀이 해결

### 기존 코드 (String 비교)

```java
if(idx >= 10) {
    Long total = 0L;

    an[idx] = liCnt;
    if(lion - apeach >= chai) {
        String a = "";
        for(int i = an.length - 1; i >= 0; i--) {
            a += an[i];
        }
        total = Long.parseLong(a);
        if(lion - apeach == chai) {
            if(total < sum) return;
        }
        for(int i = 0; i < 11; i++) {
            answer[i] = an[i];
        }
        sum = total;
        chai = lion - apeach;
    }
    return;
}
```

&nbsp;

### 수정한 코드 (배열 직접 비교)

```java
if(idx >= 10) {
    an[idx] = liCnt;
    int diff = lion - apeach;
    if(diff >= chai) {
        if(diff == chai) {
            for(int i = 10; i >= 0; i--) {
                if(an[i] > answer[i]) break;
                else if(an[i] < answer[i]) return;
            }
        }
        for(int i = 0; i < 11; i++) answer[i] = an[i];
        chai = diff;
    }
    return;
}
```

&nbsp;

### 수정한 점

| 기존 | 수정 |
|---|---|
| String으로 이어붙여 Long 비교 | `int[]` 배열에서 바로 대소 비교 |
| 화살 수 두 자리 이상이면 비교 오류 | 자릿수 관계없이 정확한 비교 가능 |
| `lion - apeach` 반복 계산 | `diff` 변수로 저장하여 재사용 |

---

&nbsp;

## ⚙️ 전체 코드

```java
import java.util.*;

class Solution {
    int[] info;
    int[] answer = new int[11];
    int chai = 0;
    boolean found = false;

    public int[] solution(int n, int[] info) {
        this.info = info;
        score(0, 0, 0, n, new int[11]);

        return found ? answer : new int[]{-1};
    }

    void score(int lion, int apeach, int idx, int liCnt, int[] an) {
        if(idx >= 10) {
            an[idx] = liCnt;
            int diff = lion - apeach;
            if(diff > 0 && diff >= chai) {
                if(diff == chai) {
                    for(int i = 10; i >= 0; i--) {
                        if(an[i] > answer[i]) break;
                        else if(an[i] < answer[i]) return;
                    }
                }
                for(int i = 0; i < 11; i++) answer[i] = an[i];
                chai = diff;
                found = true;
            }
            return;
        }

        int apCnt = info[idx];
        int sc = 10 - idx;

        // 라이언이 이 점수를 가져가는 경우
        if(liCnt > apCnt) {
            an[idx] = apCnt + 1;
            score(lion + sc, apeach, idx + 1, liCnt - apCnt - 1, an);
            an[idx] = 0;
        }

        // 라이언이 이 점수를 포기하는 경우
        if(apCnt > 0) {
            score(lion, apeach + sc, idx + 1, liCnt, an);
        } else {
            score(lion, apeach, idx + 1, liCnt, an);
        }
    }
}
```

---

&nbsp;

> 📌 **핵심 정리** : 완전탐색 문제에서 "최적해 갱신 조건"을 정확히 세우는 게 핵심이다. 점수 차이(`diff`) 비교가 1순위, 같을 때 낮은 점수 우선 비교가 2순위. 비교 로직은 String 변환보다 배열 직접 비교가 안전하다.