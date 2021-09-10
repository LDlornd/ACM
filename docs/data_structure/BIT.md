### 树状数组基础

支持单点修改与求区间和，其中区间和采用前缀和相减的方式。

??? 树状数组基础
    ```cpp
    #include <cstdio>

    typedef long long int ll;

    const int MAXN = 5e5 + 5;

    int n, m, opt, x, y;
    int arr[MAXN];
    ll sum[MAXN], BIT[MAXN];

    int lowbit(int xx) {
        return xx & -xx;
    }

    void update(int place, int delta) {
        for (; place <= n; place += lowbit(place)) {
            BIT[place] += delta;
        }
    }

    int query(int place) {
        int tans = 0;
        for (; place >= 1; place -= lowbit(place)) {
            tans += BIT[place];
        }
        return tans;
    }

    int main() {
        scanf("%d%d", &n, &m);
        for (int i = 1; i <= n; ++i) {
            scanf("%d", &arr[i]);
            sum[i] = sum[i - 1] + arr[i];
        }
        while (m--) {
            scanf("%d%d%d", &opt, &x, &y);
            if (opt == 1) update(x, y);
            else {
                printf("%lld\n", query(y) - query(x - 1) + sum[y] - sum[x - 1]);
            }
        }
        return 0;
    }
    ```