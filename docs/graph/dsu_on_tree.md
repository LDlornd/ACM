### 树上启发式合并（dsu on tree）

基于树链剖分，在统计答案时，分以下三步：

1. 递归求解轻儿子，消除轻儿子对答案的影响。
2. 递归求解重儿子，不消除其影响。
3. 遍历以亲儿子为根的子树，统计所有节点。

模板求解的是以每个节点为根的子树中，出现次数最多的颜色的编号之和。

??? "dsu on tree"
    ```cpp
    #include <cstdio>
    #include <vector>

    const int MAXN = 1e5 + 5;

    int n, u, v;
    int color[MAXN];
    int fa[MAXN], depth[MAXN];
    int sz[MAXN], hson[MAXN];
    int cnt[MAXN], max_cnt;
    long long int tans, ans[MAXN];
    int L[MAXN], R[MAXN], arr[MAXN], dfn;

    std::vector<int> con[MAXN];

    void get_tree(int cur, int father) {
        L[cur] = ++dfn;
        arr[dfn] = cur;
        fa[cur] = father;
        depth[cur] = depth[father] + 1;
        sz[cur] = 1;
        for (int nxt : con[cur]) {
            if (nxt != father) {
                get_tree(nxt, cur);
                sz[cur] += sz[nxt];
                if (hson[cur] == 0 || sz[nxt] > sz[hson[cur]]) hson[cur] = nxt;
            }
        }
        R[cur] = dfn;
    }

    void add(int cur) {
        ++cnt[color[cur]];
        if (cnt[color[cur]] > max_cnt) {
            max_cnt = cnt[color[cur]];
            tans = color[cur];
        } else if (cnt[color[cur]] == max_cnt) {
            tans += color[cur];
        }
    }

    void del(int cur) {
        --cnt[color[cur]];
    }

    void dsu(int cur, int father, bool keep) {
        // 先递归轻儿子，消除影响
        for (int nxt : con[cur]) {
            if (nxt != father && nxt != hson[cur]) dsu(nxt, cur, false);
        }
        // 递归重儿子，不消除影响
        if (hson[cur]) dsu(hson[cur], cur, true);
        // 将轻子树上的节点统计进来
        for (int nxt : con[cur]) {
            if (nxt == father || nxt == hson[cur]) continue;
            for (int i = L[nxt]; i <= R[nxt]; ++i) {
                add(arr[i]);
            }
        }
        // 统计自己
        add(cur);
        ans[cur] = tans;
        // 消除影响
        if (!keep) {
            for (int i = L[cur]; i <= R[cur]; ++i) {
                del(arr[i]);
            }
            max_cnt = 0;
            tans = 0;
        }
    }

    int main() {
        scanf("%d", &n);
        for (int i = 1; i <= n; ++i) scanf("%d", &color[i]);
        for (int i = 1; i < n; ++i) {
            scanf("%d%d", &u, &v);
            con[u].push_back(v);
            con[v].push_back(u);
        }
        get_tree(1, 0);
        dsu(1, 0, true);
        for (int i = 1; i <= n; ++i) {
            printf("%lld ", ans[i]);
        }
        return 0;
    }
    ```