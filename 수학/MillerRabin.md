[폴라드 로](/수학/PollardRho.md)   
[카테고리](/README.md)
## Miller Rabin
```cpp
inline ll multiply(ll a, ll b, ll mod) { return __int128(a) * b % mod; }

ll power(ll a, ll n, ll mod) { //a ^ n % mod
    ll res = 1;

    while (n) {
        if (n & 1) res = multiply(res, a, mod);
        a = multiply(a, a, mod);
        n >>= 1;
    }

    return res;
}

bool isPrime(ll n) { // 밀러-라빈, O(7 log^3(N) )
    if (n <= 1) return false;

    ll d = n - 1, r = 0;
    while (~d & 1) d >>= 1, ++r;

    auto check = [&](ll a) {
        ll remain = power(a, d, n);
        if (remain == 1 || remain == n - 1) return true;
        
        for (int i = 1; i < r; i++) {
            remain = multiply(remain, remain, n);
            if (remain == n - 1) return true;
        }

        return false;
    };

    vector<ll> nums = {2,325, 9375, 28178, 450775, 9780504, 1795265022};
    for (ll a : nums) if (a % n && !check(a)) return false;
    return true;
}
```
### 시간복잡도 
$O(log^3{N})$   
정확히는 $O(k~log^3{N})$   
k는 알고리즘 반복 횟수, 즉 코드에서 nums벡터의 크기(ull범위까진 k=7로 정확한 판별 가능)   

n이 매우 큰 수일 때 큰 수 곱셈의 시간복잡도를 고려해 $log^2N$를 붙인 것이라 함, ll범위에선 그냥 $O(k~logN)$으로 생각해도 되는 듯   

### 문제
[아파트 임대](https://www.acmicpc.net/problem/5615)

### 참고링크
https://www.teferi.net/ps/%EC%86%8C%EC%88%98_%ED%8C%90%EB%B3%84

### 원리
$N$이 소수이면 $~~~~~~~~~~\rightarrow$ proposition p   
$a^{N-1}\equiv1(mod~N) ~~(\because Fermat’s~~little~~theorem$)   
let) $N - 1 = d \times 2^r, ~~d \equiv 1(mod 2)$   
$a^{d \times 2^r} \equiv 1(mod~N)$   
$a^{d \times 2^{r-1}} \equiv \pm1(mod~N)$   
$a^{d \times 2^{r-1}} \equiv +1(mod~N)$ 인 경우에 대해 재귀적으로 $\pm1$로 분해하면   
$\therefore a^{d} \equiv \pm1(mod~N)~~~~~or~~~~~a^{d \times 2^i} \equiv -1 (mod~N)$  $~~~~~~~~~~\rightarrow$ proposition q   

p -> q 의 대우 ~q -> ~p 를 사용해서 소수가 아닌 수들을 거를 수 있음   
a의 값을 7개의 수 {2,325, 9375, 28178, 450775, 9780504, 1795265022}로 하여 확인할 경우 unsigned long long 범위까진 전부 걸러낼 수 있음이 알려짐   
int 범위는 {2, 7, 61}만으로도 충분함