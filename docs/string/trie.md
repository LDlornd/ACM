#### Trie 树实现

支持字符串的插入和查找操作。

??? "基本 trie 树实现"
    ```cpp
    #include <cstdio>
    #include <cstring>

    const int MAXN = 5e5 + 5;

    int n, m;
    char s[55];
    int tr[MAXN][26], flg[MAXN], tot;

    void ins(char *ss, int len) {
        int cur = 0;
        for (int i = 1; i <= len; ++i) {
            int nxt = ss[i] - 'a';
            if (!tr[cur][nxt]) tr[cur][nxt] = ++tot;
            cur = tr[cur][nxt];
        }
        flg[cur] = 1;
    }

    int query(char *ss, int len) {
        int cur = 0;
        for (int  i = 1; i <= len; ++i) {
            int nxt = ss[i] - 'a';
            if (!tr[cur][nxt]) return 0;
            cur = tr[cur][nxt];
        }
        int ret = flg[cur];
        if (flg[cur] == 1) flg[cur] = 2;
        return ret;
    }

    int main() {
        scanf("%d", &n);
        while (n--) {
            scanf("%s", s + 1);
            ins(s, strlen(s + 1));
        }
        scanf("%d", &m);
        while (m--) {
            scanf("%s", s + 1);
            int tmp = query(s, strlen(s + 1));
            if (tmp == 0) printf("WRONG\n");
            if (tmp == 1) printf("OK\n");
            if (tmp == 2) printf("REPEAT\n");
        }
        return 0;
    }
    ```