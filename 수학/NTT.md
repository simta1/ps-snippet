[카테고리](/README.md)
## NTT (Number Theoretic Transform)
```cpp
template <ll mod, ll primitiveRoot>
class NTT {
private:
    ll multiply(ll a, ll b) {
        return __int128(a) * b % mod;
    }

    ll power(ll a, ll n) { //a ^ n % mod
        ll res = 1;

        while (n) {
            if (n & 1) res = multiply(res, a);
            a = multiply(a, a);
            n >>= 1;
        }

        return res;
    }

    void ntt(vector<ll> &f, bool is_reverse) {
        int n = f.size();

        for (int i = 1, j = 0; i < n; i++) {
            int bit = n / 2;
            while (j >= bit) {
                j -= bit;
                bit /= 2;
            }
            j += bit;
            if (i < j) swap(f[i], f[j]);
        }

        vector<ll> root(n / 2, 1);
        ll w = power(primitiveRoot, (mod - 1) / n);
        if (is_reverse) w = modInv(w);
        for (int i = 1; i < root.size(); i++) root[i] = multiply(root[i - 1], w);
        
        for (int m = 2; m <= n; m *= 2) {
            int step = n / m;
            for (int i = 0; i < n; i += m) {
                for (int j = 0; j < m / 2; j++) {
                    ll even = f[i + j];
                    ll odd = multiply(root[step * j], f[i + j + m / 2]);
                    f[i + j] = (even + odd) % mod;
                    f[i + j + m / 2] = (even - odd) % mod;
                    if (f[i + j + m / 2] < 0) f[i + j + m / 2] += mod;
                }
            }
        }

        if (is_reverse) {
            ll oon = modInv(n); // one over n
            for (auto &e : f) e = multiply(e, oon);
        }
    }
    
    int powGE(int n) {
        return 1 << 32 - __builtin_clz(n) - !(n & ~-n);
    }

public:
    NTT() {
        // assert(isPrime(mod) && isPrimitiveRoot(primitiveRoot, mod))
    }

    ll modInv(ll a) {
        return power(a, mod - 2);
    }
    
    template <typename T>
    vector<ll> multiply(const vector<T> &v1, const vector<T> &v2) {
        vector<ll> a(v1.begin(), v1.end());
        vector<ll> b(v2.begin(), v2.end());

        int n = powGE(a.size() + b.size());

        a.resize(n);
        b.resize(n);
        
        ntt(a, false);
        ntt(b, false);

        for (int i = 0; i < n; i++) a[i] = multiply(a[i], b[i]);
        ntt(a, true);

        return a;
    }
};
```
### 소수, 원시근 예시
```cpp
NTT<998'244'353, 3> ntt;
NTT<2'281'701'377, 3> ntt;
NTT<2'483'027'969, 3> ntt;
```
### NTT + CRT (mod 5'665'528'335'996'813'313 ~= 5e18)
```cpp
template <typename T>
vector<ll> multiply(const vector<T> &v1, const vector<T> &v2) {
    const ll p1 = 2'281'701'377;
    const ll p2 = 2'483'027'969;

    NTT<p1, 3> ntt1;
    NTT<p2, 3> ntt2;

    vector<ll> conv1 = ntt1.multiply(v1, v2);
    vector<ll> conv2 = ntt2.multiply(v1, v2);
    
    ll i1 = ntt2.modInv(p1);
    ll i2 = ntt1.modInv(p2);

    vector<ll> res(conv1.size());
    for (int i = 0; i < res.size(); i++) { // CRT
        res[i] = (__int128(conv1[i]) * p2 * i2 + __int128(conv2[i]) * p1 * i1) % (p1 * p2);
    }
    return res;
}
```
### 시간복잡도 
$O(N~logN)$   

### 문제
[씽크스몰](https://www.acmicpc.net/problem/11385) - NTT + CRT   