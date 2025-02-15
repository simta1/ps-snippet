[카테고리](/README.md)
## 볼록 껍질(Convex Hull)
### [기하학 헤더](/기하학/Geometry%20Header.md)
<details>
<summary>Point, Cross Product, CCW</summary>

```cpp
template <typename T>
struct Point {
    T x, y;
    
    bool operator<(const Point &other) const { return tie(x, y) < tie(other.x, other.y); }
    Point operator-(const Point &other) const { return {x - other.x, y - other.y}; }
};

template <typename T>
T crossProduct(const Point<T> &p1, const Point<T> &p2) {
    return (p1.x * p2.y - p2.x * p1.y);
}

template <typename T>
int ccw(const Point<T> &p1, const Point<T> &p2, const Point<T> &p3) { // -1 : 시계, 0 : 일직선, 1 : 반시계
    T cp = crossProduct(p2 - p1, p3 - p1);
    return (cp > 0) - (cp < 0);
}
```
</details>

### Andrew's Algorithm (Monotone Chain)
```cpp
template <typename T>
vector<Point<T> > getConvexHull(vector<Point<T> > points) { // points 원본 배열 바껴도 괜찮으면 &points로 받기
    assert (points.size() >= 3);
    
    sort(points.begin(), points.end());

    vector<Point<T> > upp;
    for (auto &point : points) {
        while (upp.size() >= 2 && ccw(upp[upp.size() - 2], upp[upp.size() - 1], point) >= 0) upp.pop_back();
        upp.push_back(point);
    }

    vector<Point<T> > low;
    for (auto &point : points) {
        while (low.size() >= 2 && ccw(low[low.size() - 2], low[low.size() - 1], point) <= 0) low.pop_back();
        low.push_back(point);
    }
    
    for (int i = upp.size() - 2; i > 0; i--) low.push_back(upp[i]); // upp과 low의 시작점, 끝점은 중복되는 동일한 점임
    return low;
}
```
### 시간복잡도
$O(N~logN)$   

### 구현 주의사항
윗껍질은 시계방향이므로 ccw=-1이 될때만 push하기 위해 ccw>=0인 경우 pop을 하고,   
아래껍질은 반시계방향이므로 ccw=1이 될때만 push하기 위해 ccw<=0인 경우 pop   

### 사용설명
`getConvexHull()`이 리턴하는 다각형에서 점들은 반시계 방향 정렬돼 있음   

original points가 바껴도 된다면 `vector<Point<T> > &points`로 참조 사용   

볼록껍질만 구하는 상황이라면 [Graham Scan](/기하학/볼록껍질(Graham%20Scan).md)을 써도 상관없음   

Convex Layers를 구해야 한다면 정렬을 한번만 해도 되는 Monotone Chain이 훨씬 빠름-> $O(N~logN + KN)$   
Convex Layers를 $O(N~logN)$에 구하는 알고리즘도 있지만 이건 논외

### 문제
[볼록 껍질](https://www.acmicpc.net/problem/1708)   

### 참고링크
https://deepdata.tistory.com/992   
https://00ad-8e71-00ff-055d.tistory.com/101   