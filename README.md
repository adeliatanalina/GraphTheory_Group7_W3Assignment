# Theory Graph - Week 03 Assignment

**Group 7**
| Name | NRP | Class |
| ---- | --- | ----- |
| Nuha Usama Okbah | 5025241005 | IUP |
| Embun Nabila Rasendriya Az Zahra  | 5025241009 | IUP |
| Almira Nayla Felisitha  | 5025241014 | IUP |
| Adelia Tanalina Yumna  | 5025241078 | IUP |

## 1. Fleury's Algorithm
Fleur Algorithm is an algorithm to find an Eulerian Path, which is a path that traverses every edge of a connected graph exactly once. This algorithm avoids crossing a bridge edge (one that would split the graph if removed) unless it's the only edge left to follow.

**A. Code**
```c
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

void removeEdge(vector<int> graph[], int u, int v) {
    graph[u].erase(find(graph[u].begin(), graph[u].end(), v));
    graph[v].erase(find(graph[v].begin(), graph[v].end(), u));
}

void dfs(int u, vector<int> graph[], vector<bool> &visited) {
    visited[u] = true;
    for (int neighbor : graph[u]) {
        if (!visited[neighbor]) {
            dfs(neighbor, graph, visited);
        }
    }
}

bool isBridge(int u, int v, vector<int> graph[], int totalNodes) {
    if (graph[u].size() == 1) return false; 

    vector<bool> visited(totalNodes, false);
    int before = 0;
    dfs(u, graph, visited);
    for (bool x : visited) if (x) before++;

    removeEdge(graph, u, v);

    fill(visited.begin(), visited.end(), false);
    int after = 0;
    dfs(u, graph, visited);
    for (bool x : visited) if (x) after++;

    graph[u].push_back(v);
    graph[v].push_back(u);

    return before > after;
}

void fleuryStep(int u, vector<int> graph[], vector<vector<int>> &trail, int totalNodes) {
    for (int i = 0; i < graph[u].size(); ++i) {
        int v = graph[u][i];
        if (!isBridge(u, v, graph, totalNodes)) {
            trail.push_back({u, v});
            removeEdge(graph, u, v);
            fleuryStep(v, graph, trail, totalNodes);
            break;
        }
    }
}

vector<vector<int>> fleury(int totalNodes, vector<int> graph[]) {
    int start = 0;
    for (int i = 0; i < totalNodes; i++) {
        if (graph[i].size() % 2 != 0) {
            start = i;
            break;
        }
    }

    vector<vector<int>> trail;
    fleuryStep(start, graph, trail, totalNodes);
    return trail;
}

int main() {
    int n, m;
    cout << "Enter number of vertices and edges: ";
    cin >> n >> m;

    vector<int> graph[n];
    cout << "Enter edges (u v):" << endl;
    for (int i = 0; i < m; i++) {
        int u, v;
        cin >> u >> v;
        graph[u].push_back(v);
        graph[v].push_back(u);
    }

    int oddCount = 0;
    for (int i = 0; i < n; i++) {
        if (graph[i].size() % 2 != 0) {
            oddCount++;
        }
    }

    if (!(oddCount == 0 || oddCount == 2)) {
        cout << "Euler path not found" << endl;
        return 0;
    }

    vector<vector<int>> result = fleury(n, graph);

    cout << "Euler path:" << endl;
    for (auto &edge : result) {
        cout << edge[0] << " -> " << edge[1] << endl;
    }

    return 0;
}

```

**B. Explanation**
1) Graph representation
    ```c
    vector<int> graph[n];
    ```
    - The graph is stored as an adjacency list.
    - `graph[u]` contains all neighbors of vertex `u`.

2) Remove edge
    ```c
    void removeEdge(vector<int> graph[], int u, int v) { ... }
    ```
    - Deletes the edge `(u, v)` from both sides.
    - Because the graph is undirected, `u → v` and `v → u` are both removed. 

3) DFS to check connectivity
    ```c
    void dfs(int u, vector<int> graph[], vector<bool> &visited) { ... }
    ```
    - Visits all vertices reachable from `u`.
    - Used to check whether removing an edge breaks connectivity.

4) Check if edge is a bridge
    ```c
    bool isBridge(int u, int v, vector<int> graph[], int totalNodes) { ... }
    ```
    - An edge is a bridge if removing it disconnects the graph.
    - If `(u, v)` is the only edge from `u`, it’s not treated as a bridge (it must be used).  

5) Fleury step (recursive traversal)
    ```c
    void fleuryStep(int u, vector<int> graph[], vector<vector<int>> &trail, int totalNodes) { ... }
    ```
    - Chooses the next valid edge from vertex `u`.
    - Avoids bridges unless there’s no other choice.
    - Records the edge into trail and continues recursively.

6) Fleury’s Algorithm
    ```c
    vector<vector<int>> fleury(int totalNodes, vector<int> graph[]) { ... }
    ```
    - Finds a starting point:
        - If there are odd vertices → start at one of them.- Otherwise, start at `0` (Eulerian circuit).
    - Calls `fleuryStep` to collect the full Euler path/circuit.

7) Euler condition check
    ```c
    int oddCount = 0;
    for (int i = 0; i < n; i++) {
        if (graph[i].size() % 2 != 0) {
            oddCount++;
        }
    }

    if (!(oddCount == 0 || oddCount == 2)) {
        cout << "Euler path/circuit not found" << endl;
        return 0;
    }
    ```
    - Before running Fleury’s algorithm, the program checks the degrees of all vertices.
    - If the graph has:
        - 0 odd-degree vertices → Eulerian circuit exists
        - 2 odd-degree vertices → Eulerian path exists.
        - Otherwise, print "Euler path not found" and stop.

8) Main function
    ```c
    int main() {
    cin >> n >> m;
    ...
    vector<vector<int>> result = fleury(n, graph);
    }
    ```
    - Reads number of vertices and edges.
    - Builds the adjacency list.
    - Runs Fleury’s algorithm.
    - If valid, runs Fleury’s algorithm and prints the Euler path step by step.



**C. Input-Output Sample**

**Input**
```
Enter number of vertices and edges: 4 4
Enter edges (u v):
0 1
1 2
2 0
0 3
```

**Output**
```
Euler path:
0 -> 1
1 -> 2
2 -> 0
0 -> 3
```






## 2. Hierholzer's Algorithm


**A. Code**
```c
#include <iostream>
#include <vector>
#include <stack>
#include <algorithm>
using namespace std;

const int MAX = 100;
int n, m;
vector<int> adj[MAX];

void addEdge(int u, int v) {
    adj[u].push_back(v);
    adj[v].push_back(u);
}

int degree(int u) {
    return adj[u].size();
}

vector<int> hierholzer(int start) {
    stack<int> st;
    vector<int> path;
    st.push(start);

    while (!st.empty()) {
        int u = st.top();
        if (!adj[u].empty()) {
            int v = adj[u].back();
            adj[u].pop_back();
            // remove the reverse edge v->u as well
            for (int i = 0; i < adj[v].size(); i++) {
                if (adj[v][i] == u) {
                    adj[v].erase(adj[v].begin() + i);
                    break;
                }
            }
            st.push(v);
        } else {
            path.push_back(u);
            st.pop();
        }
    }
    reverse(path.begin(), path.end());
    return path;
}

int main() {
    cout << "Enter the number of vertices and edges: ";
    cin >> n >> m;

    cout << "Enter edges (u v):" << endl;
    for (int i = 0; i < m; i++) {
        int u, v;
        cin >> u >> v;
        addEdge(u, v);
    }

    int odd = 0, start = 0;
    for (int i = 0; i < n; i++) if (degree(i) % 2) { odd++; start = i; }

    if (!(odd == 0 || odd == 2)) {
        cout << "Euler path not found" << endl;
        return 0;
    }

    vector<int> path = hierholzer(start);
    cout << "Hierholzer path:" << endl;
    for (int v : path) cout << v << " ";
    cout << endl;
}

```

**B. Explanation**

1) Graph representation
    ```c
    vector<int> adj[MAX];
    ```
- The graph is stored as an adjacency list.
- `adj[u]` holds all neighbors of vertex `u`.


2) Adding edges
    ```c
    void addEdge(int u, int v) {
        adj[u].push_back(v);
        adj[v].push_back(u);
    }
    ```
- Adds an undirected edge between `u` and `v`.


3) Degree
    ```c
    int degree(int u) {
        return adj[u].size();
    }
    ```
- Counts how many edges connect to vertex `u`.


4) Hierholzer's Algorithm
    ```c
    vector<int> hierholzer(int start) {
        stack<int> st;
        vector<int> path;
        st.push(start);

        while (!st.empty()) {
            int u = st.top();
            if (!adj[u].empty()) {
                int v = adj[u].back();
                adj[u].pop_back();
                // also remove the reverse edge v -> u
                for (int i = 0; i < adj[v].size(); i++) {
                    if (adj[v][i] == u) {
                        adj[v].erase(adj[v].begin() + i);
                        break;
                    }
                }
                st.push(v);
            } else {
                path.push_back(u);
                st.pop();
            }
        }
        reverse(path.begin(), path.end());
        return path;
    }
    ```
    - Start at a vertex `start`.
    - If the current vertex `u` still has unused edges, take one edge `(u, v)` and push `v` on the stack.
    - At the same time, remove that edge from both sides (so it won’t be reused).
    - If the current vertex has no edges left, it becomes part of the Eulerian path → add it to path and pop it from the stack.
    - Repeat until the stack is empty.
    - Reverse the path at the end (because it’s built backwards).
    - This ensures all edges are used exactly once.


5) Main Function
    ```c
        int main() {
        cin >> n >> m;
        for (int i = 0; i < m; i++) { ... }

        int odd = 0, start = 0;
        for (int i = 0; i < n; i++) if (degree(i) % 2) { odd++; start = i; }

        if (!(odd == 0 || odd == 2)) {
            cout << "euler path not found" << endl;
            return 0;
        }

        vector<int> path = hierholzer(start);
        for (int v : path) cout << v << " ";
    }
    ```
- Read number of vertices n and edges m.
- Input all edges into adjacency lists.
- Count vertices with odd degree:
    - If 0 odd vertices → Eulerian Circuit exists.
    - If 2 odd vertices → Eulerian Path exists.
    - Otherwise, no Euler path.
- Choose the start vertex:
    - If there are 2 odd vertices, start from one of them.
    - If none are odd, start anywhere.
- Call hierholzer(start) to get the Eulerian path.
- Print the sequence of vertices.


**C. Input-Output Sample**

**Input**
```
Enter the number of vertices and edges: 6 7
Enter edges (u v):
0 1
1 2
2 0
2 3
3 4
4 5
5 3
```
**Output**
```
Hierholzer path:
3 5 4 3 2 0 1 2
```
