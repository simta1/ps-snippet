[카테고리](/README.md)
## Offline Incremental SCC
### [Disjoint Set](/자료구조/기타/Disjoint%20Set.md)
<details>
<summary>find()만 접근제어자 public으로 변경</summary>

```cpp
class DisjointSet {
private:
    vector<int> parent;

public:
    DisjointSet(int n) : parent(n + 1) {
        iota(parent.begin(), parent.end(), 0);
    }

    int find(int a) {
        if (parent[a] != a) parent[a] = find(parent[a]);
        return parent[a];
    }

    void merge(int a, int b) {
        parent[find(a)] = find(b);
    }

    bool isConnected(int a, int b) {
        return find(a) == find(b);
    }
};
```
</details>

### [SCC](/그래프%20이론/그래프/SCC.md)
TODO 예전 SCC 코드 기준임.. 기존 SCC 코드를 다시 짜서 이젠 이렇게 안 쓰는 중이긴 함..   
<details>
<summary> scc를 계산할 노드들만 따로 저장하기 위한 inGraph 선언<br>
addEdge()랑 clear()에 inGraph업데이트하는 과정 추가<br>
findSCC()에서 visited, sccNumber매번 초기화하도록 추가   
</summary>

```cpp
class Graph {
private:
    int n, dfsn, sccn;
    vector<vector<int> > adj;
    vector<int> visited, sccNumber;
    stack<int> s;

    vector<int> inGraph;

    int dfs(int cur) {
        int res = visited[cur] = dfsn++;
        s.push(cur);

        for (int next : adj[cur]) {
            if (!~visited[next]) res = min(res, dfs(next));
            else if (!~sccNumber[next]) res = min(res, visited[next]);
        }

        if (res == visited[cur]) {
            while (1) {
                int node = s.top();
                s.pop();
                sccNumber[node] = sccn;
                if (node == cur) break;
            }
            ++sccn;
        }

        return res;
    }

public:
    Graph(int n) : n(n), adj(n + 1), visited(n + 1, -1), sccNumber(n + 1, -1) {}

    void addEdge(int u, int v) {
        inGraph.push_back(u);
        inGraph.push_back(v);
        adj[u].push_back(v);
    }

    void clear() {
        for (auto i : inGraph) adj[i].clear();
        inGraph.clear();
    }

    void findSCC() {
        for (int i : inGraph) sccNumber[i] = visited[i] = -1;
        dfsn = sccn = 0;

        for (int i : inGraph) if (!~visited[i]) {
            dfs(i);
        }
    }

    int isSameSCC(int u, int v) {
        return sccNumber[u] == sccNumber[v];
    }
};
```
</details>

### Offline Incremental SCC
```cpp
vector<vector<pair<int, int> > > getFirstMergeTime(int n, const vector<pair<int, int> > &edges) {
    DisjointSet ds(n);
    Graph graph(n);

    vector<vector<pair<int, int> > > res(edges.size());

    function<void(int, int, vector<int>&)> dnc = [&](int s, int e, const vector<int> &idxes) {
        if (s == e) {
            if (s == edges.size()) return;

            for (int idx : idxes) {
                auto [u, v] = edges[idx];
                ds.merge(u, v);
                res[s].push_back({u, v});
            }
            return;
        }

        int m = s + e >> 1;
        graph.clear();
        for (int idx : idxes) if (idx <= m) {
            auto [u, v] = edges[idx];
            graph.addEdge(ds.find(u), ds.find(v));
        }
        graph.findSCC();

        vector<int> lIdxes, rIdxes;
        for (int idx : idxes) {
            auto [u, v] = edges[idx];
            if (idx <= m && graph.isSameSCC(ds.find(u), ds.find(v))) lIdxes.push_back(idx);
            else rIdxes.push_back(idx);
        }

        dnc(s, m, lIdxes);
        dnc(m + 1, e, rIdxes);
    };

    vector<int> idxes(edges.size());
    iota(idxes.begin(), idxes.end(), 0);
    dnc(0, edges.size(), idxes);

    return res;
}
```
### 시간복잡도 
$O((V+E)~logE)$   
E는 전체 간선 개수   

### 구현 주의사항
dnc(0, idxes.size() - 1, idxes)로 호출하면 안 됨.   
사이클을 이루지 않는 간선들은 계속 right로 몰리게 되므로 s == e == edges.size() - 1일 때 실제로 합쳐지는 간선과 그렇지 않은 간선을 구별할 수 없음   
따로 dnc(0, idxes.size(), idxes)로 호출해서 s == e == edges.size()일 때의 간선들만 제외해주면 됨   

findSCC()에서 매번 inGraph의 노드들에 대해서 -1로 초기화해줘야 됨. 유니온파인드로 압축하면서 같은 인덱스를 다시 확인하게 되기 때문   

self-loop나 parallel edge 있어도 잘 작동함

### 사용설명
getFirstMergeTime(n, edges)는 edges의 각 간선들이 처음으로 사이클에 포함되는 순간을 계산해서 리턴

ex)
edges = {{1, 2}, {2, 3}, {2, 1}}로 주어진다면
0, 1번째에는 사이클이 생기지 않고, edges[2]가 추가될 때 edges[0]과 edges[2]가 처음으로 사이클에 포함되게 되므로
result[2]에 edges[0]과 edges[2]의 정보가 저장됨

사용예시 ([Link Cut Digraph](https://www.acmicpc.net/problem/19028))   
main함수 안에서 추가로 DisjointSet 선언해서 scc합쳐가며 오프라인 쿼리로 풀면 된다.   
dnc함수 안에서 s==e일 때마다 ans업데이트하고 출력하는 식으로도 풀 수 있긴 한데 코드 분리를 위해 따로 계산했다.   
```cpp
int main() {
    cin.tie(0) -> sync_with_stdio(0);

    int n, m;
    cin >> n >> m;

    vector<pair<int, int> > edges(m);
    for (auto &[u, v] : edges) cin >> u >> v;

    auto newMergedNodes = getFirstMergeTime(n, edges);

    DisjointSet ds(n);
    vector<int> sz(n + 1, 1);
    ll ans = 0;
    for (int i = 0; i < m; i++) {
        for (auto [u, v] : newMergedNodes[i]) {
            u = ds.find(u);
            v = ds.find(v);
            if (u == v) continue;

            ans += ll(sz[u]) * sz[v];
            ds.merge(u, v);
            sz[v] += sz[u];
        }

        cout << ans << "\n";
    }
    
    return 0;
}
```

### 문제
[Link Cut Digraph](https://www.acmicpc.net/problem/19028)   
[F. Simultaneous Coloring](https://codeforces.com/contest/1989/problem/F)

### 참고링크
https://cheet0se.tistory.com/18   
https://codeforces.com/blog/entry/91608