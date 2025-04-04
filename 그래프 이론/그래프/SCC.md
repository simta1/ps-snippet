[Offline Incremental SCC](/그래프%20이론/그래프/Offline%20Incremental%20SCC.md)   
[카테고리](/README.md)
## SCC(Strongly Connected Component)
### Tarjan's algorithm
```cpp
pair<vector<vector<int> >, vector<int> > getSCC(int n, const vector<vector<int> > &adj) { //  auto [sccs, sccn] = getSCC(n, adj);로 사용 // sccs에는 위상정렬된 순서로 {scc1{}, scc2{}, ... } 저장되어 있음
    vector<int> dfsn(n + 1, 0), sccn(n + 1, -1);
    stack<int> s;

    int dfsi = 0, scci = 0;
    function<int(int)> dfs = [&](int cur) -> int {
        int low = dfsn[cur] = ++dfsi;
        s.push(cur);

        for (auto next : adj[cur]) if (!~sccn[next]) low = min(low, dfsn[next] ? dfsn[next] : dfs(next));

        if (low == dfsn[cur]) {
            while (1) {
                int node = s.top();
                s.pop();
                sccn[node] = scci;
                if (node == cur) break;
            }
            ++scci;
        }

        return low;
    };
    for (int i = 1; i <= n; i++) if (!dfsn[i]) dfs(i);

    vector<vector<int> > sccs(scci);
    for (int i = 1; i <= n; i++) sccs[sccn[i] = scci - 1 - sccn[i]].push_back(i);
    return {sccs, sccn}; // sccn은 0-based임에 주의, 노드 x는 sccs[sccn[x]]에 포함되어 있음
}
```
### SCC끼리 묶은 DAG 만들기
```cpp
pair<int, vector<vector<int> > > graphToDAG(int n, const vector<vector<int> > &adj, const vector<vector<int> > &sccs, const vector<int> &sccn) { // sccn은 0-based이지만 DAG의 노드는 1-based로 만들어서 리턴하니 주의
    vector<int> toEdgeExist(sccs.size() + 1, -1);
    vector<vector<int> > dag(sccs.size() + 1);

    for (auto &scc : sccs) for (auto u : scc) {
        for (auto v : adj[u]) if (sccn[u] != sccn[v] && toEdgeExist[sccn[v]] != sccn[u]) {
            toEdgeExist[sccn[v]] = sccn[u];
            dag[sccn[u] + 1].push_back(sccn[v] + 1);
        }
    }

    // for (auto &scc : sccs) for (auto u : scc) cout << "orig node " << u << " -> dag node " << sccn[u] + 1 << "\n";
    // for (int u = 1; u <= sccs.size(); u++) for (auto v : dag[u]) cout << "dag edge : " << u << " -> " << v << "\n";
    
    return {sccs.size(), dag}; // auto [dagN, dag] = graphToDAG(n, adj, sccs, sccn); 으로 사용
}
```
주석 코드로 새로 만들어진 dag의 구조를 출력할 수 있음   
sccn은 0-based로 매겨지지만 dag그래프에선 노드 번호가 1-based임에 주의   

dag에서 노드의 번호는 이미 위상정렬되어 있음(sccn이 위상정렬된 순이기 때문)   

`toEdgeExist`는 간선을 추가했는지 저장하기 위한 배열로, 변수 `sccn[u]`를 [trueValue](/C++/기타/Variable%20Name.md#truevalue)로 사용해서 구현한 것임.   
toEdgeExist를 확인하지 않고 그냥 그래프를 구성해도 문제될 것은 없지만 DAG에 parallel edge를 추가하지 않기 위해선 필요한 과정   

### 시간복잡도 
$O(V + E)$   

### 구현 주의사항
타잔 알고리즘으로 SCC를 구할 경우 위상정렬의 역순이 되므로 sccNumber의 값이 클수록 위상정렬 시 앞 쪽에 위치   

self-loop나 parallel edge있어도 잘 작동하므로 딱히 고려할 필요 없음

### 문제
[Strongly Connected Component](https://www.acmicpc.net/problem/2150)   

### graphToDAG 사용 예시코드
백준 [Spy Network](https://www.acmicpc.net/problem/10287)의 코드.   
```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;

// getSCC
// graphToDAG

int main() {
    cin.tie(0) -> sync_with_stdio(0);
    
    int TC;
    for (cin >> TC; TC--;) {
        int n, m;
        ll target;
        cin >> n >> m >> target;
        
        vector<vector<int> > adj(n + 1);
        while (m--) {
            int u, v;
            cin >> u >> v;
            adj[u].push_back(v);
        }
        
        auto [sccs, sccn] = getSCC(n, adj);
        auto [dagN, dag] = graphToDAG(n, adj, sccs, sccn);

        vector<ll> dp(sccs.size() + 1, 0);
        for (int i = 1; i <= n; i++) {
            ll x;
            cin >> x;
            dp[sccn[i] + 1] = __gcd(dp[sccn[i] + 1], x);
        }

        for (int u = 1; u <= dagN; u++) {
            for (auto v : dag[u]) dp[v] = __gcd(dp[v], dp[u]);
        }
        
        int ans = 0;
        for (int i = 1; i <= n; i++) ans += dp[sccn[i] + 1] == target;
        cout << ans << "\n";
    }
    
    return 0;
}
```

dagN == sccs.size()이므로 dp배열의 크기는 dagN+1과 sccs.size()+1 중 무엇을 선택하든 상관없다.   
처음에 dp를 초기화할 때 i노드의 값을 `dp[sccn[i] + 1]`에 누적한 이유는 `sccn[i]`가 0-based이고 dag그래프는 1-based를 사용하기 때문   
`graphToDAG()`의 결과로 얻어진 dag에서 노드의 번호는 위상정렬된 순서이므로,   
`for (int u = 1; u <= dagN; u++) for (auto v : dag[u]) dp[v] = __gcd(dp[v], dp[u]);`에서 u는 위상정렬된 순서로 들어가며 따라서 dp배열도 옳게 업데이트된다.   

### SCC 사용 예시코드
```cpp
vector<bool> isLeafNode(sccs.size(), true);
for (int u = 1; u <= n; u++) {
    for (auto v : adj[u]) if (sccn[u] != sccn[v]) isLeafNode[sccn[u]] = false;
}

vector<int> leafNodes;
for (int i = 1; i <= n; i++) if (isLeafNode[sccn[i]]) leafNodes.push_back(i);
```
SCC끼리 묶었을 때 기준 leafNode인지 확인하는 경우 굳이 `graphToDAG()`를 사용하지 않고도 간단히 짤 수 있다. `graphToDAG()`를 사용하는 게 더 나은 상황과 아닌 상황이 있으니 사용하기 전에 생각해보는 게 좋다.   