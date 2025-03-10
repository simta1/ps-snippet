[카테고리](/README.md)
## Suffix Array, LCP Array
### Suffix Array (Mander-Myers)
```cpp
vector<int> getSuffixArray(const string &st) {
    int n = st.size();
    vector<int> sa(n), rank(n), tmp(n);
    for (int i = 0; i < n; i++) sa[i] = i, rank[i] = st[i];

    vector<int> cnt(max(n, 256) + 1, 0); // 아스키코드 최대값 포함 가능하게 선언

    auto countingSort = [&](int t) {
        fill(cnt.begin(), cnt.end(), 0);
        for (int i = 0; i < n; i++) ++cnt[sa[i] + t < n ? rank[sa[i] + t] : 0];
        for (int i = 1; i < cnt.size(); i++) cnt[i] += cnt[i - 1];
        for (int i = n - 1; i >= 0; i--) tmp[--cnt[sa[i] + t < n ? rank[sa[i] + t] : 0]] = sa[i];
        swap(sa, tmp);
    };

    for (int t = 1; ; t <<= 1) {
        countingSort(t);
        countingSort(0);

        tmp[sa[0]] = 1;
        for (int i = 1; i < n; i++) tmp[sa[i]] = tmp[sa[i - 1]] +  (rank[sa[i - 1]] != rank[sa[i]] || rank[sa[i - 1] + t] != rank[sa[i] + t]);
        swap(rank, tmp);

        if (rank[sa.back()] == n) break;
    }

    return sa;
}
```
### LCP Array (Kasai's algorithm)
```cpp
vector<int> getLCPArray(const string &st, const vector<int> &sa) {
    int n = st.size();
    assert(n >= 1);

    vector<int> rank(n), lcp(n - 1);
    for (int i = 0; i < n; i++) rank[sa[i]] = i; // sa[idx]=i -> st[i:]가 idx번째 접미사 // rank[i]=idx 즉, st[i:]가 몇번째 접미사인지 rank[i]에 저장
    for (int i = 0, h = 0; i < n; ++i, h -= !!h) if (rank[i]) {
        for (int j = sa[rank[i] - 1]; j + h < n && i + h < n && st[j + h] == st[i + h];) ++h;
        lcp[rank[i] - 1] = h;
    }
    return lcp;
} // lcp[i]는 sa[i]와 sa[i + 1]의 최장 공통 접두사 // 따라서 lcp배열의 크기는 n-1임
```

### 가장 긴 반복 부분 문자열, 서로 다른 부분 문자열의 개수, k번 이상 등장하는 서로 다른 부분 문자열의 개수([Deque trick](/기타/Deque%20Trick.md) 필요), k번 이상 등장하는 가장 긴 부분 문자열([Deque trick](/기타/Deque%20Trick.md) 필요)
```cpp
string longestRepeatedSubstring(const string &st, const vector<int> &sa, const vector<int> &lcp) {
    if (st.size() == 1) return "";
    int i = max_element(lcp.begin(), lcp.end()) - lcp.begin();
    return st.substr(sa[i], lcp[i]);
}

long long countDistinctSubstrings(const string &st, const vector<int> &lcp) {
    long long n = st.size();
    return n * (n + 1) / 2 - accumulate(lcp.begin(), lcp.end(), 0LL);
}

vector<int> dequeTrickMin(int len, const vector<int> &v) { // res[i]는 v[max(0, i - len + 1)]~v[i]의 최솟값 // 즉, 정확히 길이가 len인 구간의 최솟값은 res[len-1:n)에 저장됨
    vector<int> res(v.size());
    deque<int> dq;

    for (int i = 0; i < v.size(); i++) {
        while (!dq.empty() && v[dq.back()] > v[i]) dq.pop_back();
        dq.push_back(i);
        if (i - dq.front() + 1 > len) dq.pop_front();
        res[i] = v[dq.front()];
    }
    
    return res;
}

long long countDistinctSubstringsRepeatedAtLeastK(const string &st, const vector<int> &lcp, int k) { // k번 이상 등장하는 부분 문자열 종류의 수
    assert(k >= 2); // k=1일 땐  n * (n + 1) / 2 - accumulate(lcp.begin(), lcp.end(), 0LL);
    if (st.size() < k) return 0;

    auto rmq = dequeTrickMin(k - 1, lcp); // sa배열 k개의 최장공통접두사가 필요하므로 lcp배열에선 k-1개씩 뽑아서 min값을 구하면 됨
    long long res = rmq[k - 2];
    for (int i = k - 1; i < st.size() - 1; i++) res += max(0, rmq[i] - rmq[i - 1]);
    return res;
}

string longestSubstringRepeatedAtLeastK(const string &st, const vector<int> &sa, const vector<int> &lcp, int k) { // k번 이상 등장하는 부분 문자열 중 가장 긴 부분 문자열
    if (k == 1) return st;
    if (st.size() < k) return "";

    auto rmq = dequeTrickMin(k - 1, lcp); // sa배열 k개의 최장공통접두사가 필요하므로 lcp배열에선 k-1개씩 뽑아서 min값을 구하면 됨
    int i = max_element(rmq.begin() + k - 2, rmq.end()) - rmq.begin();
    return st.substr(sa[i], rmq[i]);
}
```
### 시간복잡도
suffix array $O(N \log{N})$   
(suffix 배열이 주어져 있을 때) lcp array $O(N)$   

suffix array를 $O(N)$에 구하는 알고리즘도 있지만 복잡해서 잘 쓰지 않는 듯   

### 구현 주의사항
group의 원소는 0 초과의 값을 가져야 된다.   
`countingSort`에서 `group[a + t]`에 접근할 때 인덱스 초과하면 0으로 바꿔서 `cnt[0]`에 저장하므로 이 0과 구별하기 위해선 group배열의 값은 0을 가지면 안 된다.   
`tmp[sa[0]] = 1`로 초기화하는 것도 같은 이유다.   

### 사용설명
getLCPArray()에서 만드는 rank배열이 필요한 문제도 가끔 있음   

lcp배열에서 RMQ구하면 i번째 접미사부터 j번째 접미사까지의 최장공통접두사의 길이를 구할 수 있음   
RMQ는 전처리 $O(N\log{N})$ 쿼리 $O(1)$의 구현을 사용할 때도 있고, `countDistinctSubstringsRepeatedAtLeastK()`에서처럼 길이가 정해진 구간에 대해서만 계산하는 경우엔 Deque Trick으로 모든 구간에 대해 $O(N)$으로 계산할 수도 있음([RMQ](/기타/RMQ.md) 참고)   

접미사 순서대로 출력하고 싶으면 아래코드 사용   
```cpp
for (auto i : sa) cout << st.substr(i) << "\n";
```

### 문제
[Suffix Array](https://www.acmicpc.net/problem/9248)   
[반복 부분문자열](https://www.acmicpc.net/problem/1605) - `longestRepeatedSubstring()`   
[Repeated Substrings](https://www.acmicpc.net/problem/16415) - `longestRepeatedSubstring()`   
[서로 다른 부분 문자열의 개수 2](https://www.acmicpc.net/problem/11479) - `countDistinctSubstrings()`   
[반복되는 부분 문자열](https://www.acmicpc.net/problem/10413) - `countDistinctSubstringsRepeatedAtLeastK(k=2)`   
[Milk Patterns](https://www.acmicpc.net/problem/6206) - `longestSubstringRepeatedAtLeastK()`   
[Prefix와 Suffix](https://www.acmicpc.net/problem/13576) - rank배열 필요, lcp배열에서 rmq 계산해야 되는 문제   
[Stammering Aliens](https://www.acmicpc.net/problem/3864) - lcp에서 min rmq, sa에서 max rmq   

### 원리
rank는 sa배열에 대한 역함수. 즉, st[i:]가 sa에서 몇 번째인지 저장   
가장 긴 접미사부터 계산해가는 과정에서 필요함   
아래 코드는 모두 동치   
```cpp
h = max(h - 1, 0);
if (h) --h;
h -= bool(h);
h -= !!h;
```

### 참고링크
https://blog.naver.com/kks227/221028710658   
https://loosie.tistory.com/798   
https://blog.myungwoo.kr/57   
https://koosaga.com/125   
https://infossm.github.io/blog/2021/07/18/suffix-array-and-lcp/   
https://m.blog.naver.com/jqkt15/221795956351 - 반복 부분 문자열의 개수   