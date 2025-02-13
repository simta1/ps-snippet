[카테고리](/README.md)
## FFT (Fast Fourier Transform)
```cpp
namespace FourierTransform {
    using ll = long long;
    // using ld = long double;
    using ld = double;
    using cpx = complex<ld>;
    const ld PI = acos(-1); 

    void fft(vector<cpx> &f, bool is_reverse) {
        int n = f.size();
        
        for (int i = 1, j = 0; i < n; i++) {
            for (int bit = n >> 1; bit; bit >>= 1) {
                j ^= bit;
                if (j & bit) break;
            }
            if (i < j) swap(f[i], f[j]);
        }

        for (int m = 2; m <= n; m <<= 1) {
            ld t = 2 * PI / m * (is_reverse ? -1 : 1);
            cpx w(cos(t), sin(t));

            for (int i = 0; i < n; i += m) {
                cpx w_j(1, 0);
                for (int j = 0; j < m / 2; j++, w_j *= w) {
                    cpx even = f[i + j];
                    cpx odd = w_j * f[i + j + m / 2];
                    f[i + j] = even + odd;
                    f[i + j + m / 2] = even - odd;
                }
            }
        }

        if (is_reverse) for (auto &e : f) e /= n;
    }

    int powGE(int n) {
        return 1 << 31 - __builtin_clz(n) + !!(n & (n - 1));
    }

    template <typename T>
    vector<ll> multiply(const vector<T> &v1, const vector<T> &v2) {
        vector<cpx> a(v1.begin(), v1.end());
        vector<cpx> b(v2.begin(), v2.end());

        int n = powGE(a.size() + b.size());

        a.resize(n);
        b.resize(n);
        
        fft(a, false);
        fft(b, false);

        for (int i = 0; i < n; i++) a[i] *= b[i];
        fft(a, true);

        vector<ll> res(n);
        for (int i = 0; i < n; i++) res[i] = round(a[i].real());
        return res;
    }

    const int splitBit = 15;
    template <typename T>
    vector<ll> multiplyPrecisely(const vector<T> &v1, const vector<T> &v2) {
        int n = pow2GE(v1.size() + v2.size());

        vector<cpx> a1(n), a2(n), b1(n), b2(n);
        for (int i = 0; i < v1.size(); i++) {
            a1[i] = v1[i] & ~-(1 << splitBit);
            a2[i] = v1[i] >> splitBit;
        }
        for (int i = 0; i < v2.size(); i++) {
            b1[i] = v2[i] & ~-(1 << splitBit);
            b2[i] = v2[i] >> splitBit;
        }

        fft(a1, false);
        fft(a2, false);
        fft(b1, false);
        fft(b2, false);

        vector<cpx> c1(n), c2(n), c3(n);
        for(int i=0; i<n; i++){
            c1[i] = a1[i] * b1[i];
            c2[i] = a1[i] * b2[i] + a2[i] * b1[i];
            c3[i] = a2[i] * b2[i];
        }

        fft(c1, true);
        fft(c2, true);
        fft(c3, true);

        vector<ll> res(n);
        for (int i = 0; i < n; i++) {
            ll a = llround(c1[i].real()); // % mod;
            ll b = llround(c2[i].real()); // % mod;
            ll c = llround(c3[i].real()); // % mod;
            res[i] = (a + (b << splitBit) + (c << 2 * splitBit)); // % mod;
            // if (res[i] < 0) res[i] += mod;
        }
        return res;
    }

    template <typename T>
    vector<ll> square(const vector<T> &v) {
        vector<cpx> a(v.begin(), v.end());

        int n = powGE(a.size() * 2);

        a.resize(n);
        fft(a, false);
        for (int i = 0; i < n; i++) a[i] *= a[i];
        fft(a, true);

        vector<ll> res(n);
        for (int i = 0; i < n; i++) res[i] = round(a[i].real());
        return res;
    }
}
```
### 시간복잡도 
$O(N~logN)$   

### 구현 주의사항
실수 오차가 생각보다 커질 수 있다.   

conv할 벡터의 크기를 미리 2의 거듭제곱으로 맞춘 뒤 multiply()나 square()함수를 사용하면   
if ((n >> 1) != a.size()) n <<= 1; 코드 덕분에 메모리 효율이 꽤 개선된다. (아래 __사용관련__ 참고)

### 사용설명
오차범위 확인할 때 참고 -> [부동소숫점 오류](https://www.acmicpc.net/blog/view/37)

최대/최소값의 범위가 그리 크지 않다면 using ld = double; 사용   
long double과 double 시간 차이가 꽤 크게 난다.   
[큰 수 곱셈 (2)](https://www.acmicpc.net/problem/15576) 문제 기준으로 long double에서 688ms였던 코드가 double로 바꾸니 268ms까지 줄어듦

resize되는 크기는 벡터의 크기가 2의 거듭제곱일 때 가장 효율적(?)임    
ex) 6, 7 conv -> resize(16) // 솔직히 13까지만 알아도 될텐데 (당연히 IDFT까지 끝내고 나면 14, 15, 16번째 항은 0이 될 것임) FFT 하기 위해서 더 크게 선언함   
ex) 4, 4 conv -> resize(8) // 선언된 공간 모두 사용함   
ex) 8, 8 conv -> resize(16) // 선언된 공간 모두 사용함      
2의 거듭제곱이 아닐 때는 쓰지도 않을 공간까지 필요 이상으로 resize를 크게 함.   
평소에는 크게 상관없지만 [보석 가게](https://www.acmicpc.net/problem/13575) 같은 문제에선 multiply를 많이 해야되서 이런 비효율이 계속 쌓이면 문제가 됨.   
[보석 가게](https://www.acmicpc.net/problem/13575) 문제 기준으로 처음 벡터를 선언할 때 크기 1000으로 선언하면 생각보다 나비효과가 크게 작용해서 mle가 나는 듯 함.   
처음부터 벡터의 크기를 2의 거듭제곱이 되도록(이 문제에선 1024로) 선언한 뒤 거듭제곱을 하니 mle가 해결됐음.   

### 문제
[큰 수 곱셈 (2)](https://www.acmicpc.net/problem/15576)   
[큰 수 곱셈 (3)](https://www.acmicpc.net/problem/22289)

### 원리
```cpp
void fft(vector<cpx> &f, bool is_reverse) {
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

    // ...
}
```
위 코드는 FFT의 비재귀 구현을 위해 f[i]와 f[bitReverse(i)]를 swap한다.
아래 코드를 훨씬 효율적이고 빠르게 한 코드다.
```cpp
int bitReverse(int n, int len) {
    int rev = 0;
    for (int i = 0; i < len; i++) {
        rev |= ((n >> i) & 1) << (len - i - 1);
    }
    return rev;
}

void fft(vector<cpx> &f, bool is_reverse) {
    int n = f.size();
    int len = __builtin_ctz(n);
    
    for (int i = 0; i < n; i++) {
        int j = bitReverse(i, len);
        if (i < j) swap(f[i], f[j]);
    }

    // ...
}
```

### 참고링크
https://algoshitpo.github.io/2020/05/20/fft-ntt/   
