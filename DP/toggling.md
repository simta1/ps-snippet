[카테고리](/README.md)
## toggling
```cpp
/*
조건 :
1) dp[i]가 dp[i - 1]의 영향만 받을 때
2) 결과적으로 구하고자하는 값이 dp[n] 하나일 때 (이전 dp값들은 버려도 될 때)
*/

// 기본 코드
for (int i = 1; i < n; i++) dp[i] <- f(dp[i - 1])
cout << dp[n];

// 토글링 적용
for (int i = 1; i < n; i++) dp[i & 1] <- f(dp[i - 1 & 1])
return dp[n & 1];
```
### 공간복잡도
토글링 사용한 차원의 크기가 N일 때 사용하기 전에 비해 공간복잡도 $\dfrac{1}{N}$로 감소

### 구현 주의사항
`dp[i & 1]` 사용하기 전 초기화 필요한지 생각   
`dp[i & 1] = f(dp[i - 1 & 1])` 꼴이면 상관없겠지만   
`dp[i & 1] += f(dp[i - 1 & 1])` 꼴이면 이전에 있던 값들을 날려주고 사용해야 함   

```cpp
for (int i = 1; i < n; i++) {
    // dp[i & 1] 초기화 필요한지 확인해야 됨
    dp[i & 1] <- f(dp[i - 1 & 1])
}
```

### 문제
토글링 안 한다고 메모리초과 나는 문제는 아직 못 봄   
아래는 그냥 토글링으로 메모리 사용량을 줄일 수 있는 문제지 토글링이 필수인 문제는 아님   
[템포럴 그래프](https://www.acmicpc.net/problem/25953)   

### 참고링크
https://ps.mjstudio.net/boj-10759   