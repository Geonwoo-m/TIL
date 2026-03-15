# 체육복 - 탐욕법

## 왜 Set을 썼나?
- 특정 학생이 목록에 있는지 빠르게 확인하기위해서 Set 사용
- 배열은 O(n), Set은 O(1)

## 핵심 로직
1. 도난 + 여벌 동시인 학생 양쪽에서 제거
2. 앞번호 먼저 빌려주기
3. 전체 - 못 빌린 학생 수

## 최종 코드
```java
import java.util.*;

public class Solution {
    public int solution(int n, int[] lost, int[] reserve) {
        Set<Integer> realLost = new HashSet<>();
        Set<Integer> realReserve = new HashSet<>();

        for (int l : lost) realLost.add(l);
        for (int r : reserve) realReserve.add(r);

        for (int r : reserve) realLost.remove(r);
        for (int l : lost) realReserve.remove(l);

        List<Integer> sortedReserve = new ArrayList<>(realReserve);
        Collections.sort(sortedReserve);

        for (int r : sortedReserve) {
            if (realLost.contains(r - 1)) {
                realLost.remove(r - 1);
            } else if (realLost.contains(r + 1)) {
                realLost.remove(r + 1);
            }
        }

        return n - realLost.size();
    }
}
```

## 느낀 점
- 처음엔 어떤 자료구조를 써야할 지도 모르겠고, 초반엔 배열로만 문제를 풀기 시작,
  하지만 코드가 길어지고 시간복잡도를 생각하니 Set을 써야겠다고 생각함.
- 도난+여벌 겹치는 학생을 먼저 처리 안 하면 틀린다는 것 → 전처리 순서가 중요하다는 걸 배움.
- 앞으로 비슷한 문제에서 특정 값이 존재하는지 빠르게 확인이 필요하면 Set을 먼저 생각해야겠음.
