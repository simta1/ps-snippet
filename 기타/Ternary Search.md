[카테고리](/README.md)
### Ternary Search (min)
```cpp
auto ternarySearch = [&](int l, int r) {
    int lo = l, hi = r;
    while (hi - lo >= 3) {
        int p = (lo * 2 + hi) / 3, q = (lo + hi * 2) / 3;
        if (f(p) > f(q)) lo = p;
        else hi = q;
    }

    T ans = numeric_limits<T>::max();
    for (int i = lo; i <= hi; i++) res = min(res, f(i));
    return res;
};
```
### Ternary Search (max)
```cpp
auto ternarySearch = [&](int l, int r) {
    int lo = l, hi = r;
    while (hi - lo >= 3) {
        int p = (lo * 2 + hi) / 3, q = (lo + hi * 2) / 3;
        if (f(p) > f(q)) hi = q;
        else lo = p;
    }

    T ans = -numeric_limits<T>::max();
    for (int i = lo; i <= hi; i++) ans = max(ans, f(i));
    return res;
}
```
### 시간복잡도 
$O(log_{1.5}N)$   

### 백준문제
[Haybale Distribution](https://www.acmicpc.net/problem/31059)