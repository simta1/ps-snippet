[카테고리](/README.md)
### Extended GCD
```cpp
tuple<ll, ll, ll> extendedGCD(ll a, ll b) { // ax + by = gcd(a, b)
    if (b == 0) return {1, 0, a};

    auto [x, y, g] = extendedGCD(b, a % b);
    return {y, x - (a / b) * y, g};
}
```
### Modular Inverse
```cpp
ll modInverse(ll a, ll b) {
    auto [x, y, g] = extendedGCD(a, b); //ax + by = g
    if (g == 1) return (x + b) % b;
    return -1;
}
```
mod하는 수가 소수라면 페르마의 소정리에 따라 $a^{-1} \equiv a^{p-2} (mod~p)$로 계산할 수도 있다.   

### 시간복잡도 
$O(logN)$   

### 사용설명
modInverse(n, MOD);로 사용

### 문제
[Σ](https://www.acmicpc.net/problem/13172)

### 원리
베주항등식 $ax_1 + by_1 = g$ 는 항상 해를 가짐   
$let)~ a=bk+r$   
$(bk + r)x_1 + by_1 = g$   
$b(kx_1+y_1) + rx_1 = g$   
$let)~ x_2 = kx_1+y_1, y_2=x_1$   
$\therefore x_1=y_2, y_1 = x_2-ky_2$
