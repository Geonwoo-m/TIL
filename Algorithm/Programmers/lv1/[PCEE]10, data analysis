# 데이터 분석 - 구현

## 왜 ArrayList<int[]>를 썼나?
- 조건을 만족하는 행만 골라내야 해서 크기가 유동적인 ArrayList 사용
- 2차원 배열은 크기 고정이라 필터링 후 크기를 미리 알 수 없음
- ArrayList는 int[]의 실제 값을 복사하지 않고 **주소(참조값)** 만 저장

## 핵심 로직
1. ext 문자열을 if + equals()로 인덱스로 변환 (code=0, date=1, maximum=2, remain=3)
2. 해당 인덱스 값이 val_ext보다 작은 행만 ArrayList에 추가 (필터링)
3. sort_by 문자열도 동일하게 if + equals()로 인덱스 변환
4. 해당 인덱스 기준으로 람다를 이용해 오름차순 정렬
5. ArrayList → int[][] 변환 후 return

## 정렬 람다 이해
```java
list.sort((a, b) -> a[sortIdx] - b[sortIdx]);
```
- a, b는 list 안의 int[] 두 개를 꺼내 비교
- 반환값이 음수 → a 앞에 유지 / 양수 → b가 앞으로 (swap)
- 결과적으로 작은 값이 앞으로 오는 **오름차순 정렬**

## 최종 코드
```java
import java.util.*;

class Solution {
    public int[][] solution(int[][] data, String ext, int val_ext, String sort_by) {
        int extIdx;
        if (ext.equals("code"))         extIdx = 0;
        else if (ext.equals("date"))    extIdx = 1;
        else if (ext.equals("maximum")) extIdx = 2;
        else                            extIdx = 3;

        List<int[]> list = new ArrayList<>();
        for (int i = 0; i < data.length; i++) {
            if (data[i][extIdx] < val_ext) {
                list.add(data[i]);
            }
        }

        int sortIdx;
        if (sort_by.equals("code"))         sortIdx = 0;
        else if (sort_by.equals("date"))    sortIdx = 1;
        else if (sort_by.equals("maximum")) sortIdx = 2;
        else                                sortIdx = 3;

        list.sort((a, b) -> a[sortIdx] - b[sortIdx]);

        int[][] result = list.toArray(new int[list.size()][]);
        return result;
    }
}
```

## 느낀 점
- 처음엔 if + equals로 문자열을 하나하나 비교해서 인덱스를 구하는 방식이 길어보였지만, 로직 흐름을 직접 따라가며 이해할 수 있었음. (추후 익숙해지면 람다및 스트림으로 변경)
- ArrayList가 int[]를 복사하는 게 아니라 주소만 저장됨.
- 람다의 반환값(음수/양수)으로 정렬 순서가 결정된다는 원리를 이해 /  내림차순은 b - a로 바꾸면 됨.
