#### 递归 FFT

给定两个多项式，快速相乘。先做一遍 FFT 把给定的多项式从系数表示形式转为点值表示形式，再用  的时间将点值相乘，得到结果多项式的点值表示形式。对结果多项式做一遍逆 FFT 即可得到其系数表示形式。

由于单位根具有的特殊的性质，FFT 可以通过分治在 $\Theta(n\log n)$ 的时间内完成。

??? note "递归 FFT"
    ```cpp
    #include <cstdio>
    #include <cmath>
    #include <complex>

    typedef std::complex<double> comp;

    const double pi = acos(-1.0);
    const int MAXN = 1e6 + 5;

    int n, m, k;
    comp F[MAXN << 2], G[MAXN << 2];

    void FFT(int lmt, comp arr[], int opt) {
        if (lmt == 1) return;
        comp odd[lmt >> 1], eve[lmt >> 1];
        for (int i = 0; i < lmt; i += 2) {
            eve[i >> 1] = arr[i];
            odd[i >> 1] = arr[i + 1];
        }
        FFT(lmt >> 1, odd, opt);
        FFT(lmt >> 1, eve, opt);
        comp cur(1, 0), step(cos(2.0 * pi / lmt), opt * sin(2.0 * pi / lmt));
        for (int i = 0; i < lmt / 2; ++i, cur *= step) {
            arr[i] = eve[i] + cur * odd[i];
            arr[i + lmt / 2] = eve[i] - cur * odd[i];
        }
    }

    int main() {
        scanf("%d%d", &n, &m);
        for (int i = 0; i <= n; ++i) {
            scanf("%d", &k);
            F[i].real(k);
        }
        for (int i = 0; i <= m; ++i) {
            scanf("%d", &k);
            G[i].real(k);
        }
        int lmt = 1;
        while (lmt <= n + m) {
            lmt <<= 1;
        }
        FFT(lmt, F, 1);
        FFT(lmt, G, 1);
        for (int i = 0; i <= lmt; ++i) {
            F[i] *= G[i];
        }
        FFT(lmt, F, -1);
        for (int i = 0; i <= n + m; ++i) {
            printf("%.0lf ", fabs(F[i].real() / lmt));
        }
        return 0;
    }
    ```

#### 非递归 FFT（蝴蝶优化）

观察递归 FFT 的过程，每次都是将多项式的奇数次项和偶数次项系数分开，一直到只剩下一个数，我们来模拟一下拆分与合并的过程。假设多项式次数为 $8$ 。

<table align="center">
    <tr>
        <td colspan="8" align="center">0,1,2,3,4,5,6,7</td>
    </tr>
    <tr>
        <td colspan="4" align="center">0,2,4,6</td>
        <td colspan="4" align="center">1,3,5,7</td>
    </tr>
        <td colspan="2" align="center">0,4</td>
        <td colspan="2" align="center">2,6</td>
        <td colspan="2" align="center">1,5</td>
        <td colspan="2" align="center">3,7</td>
    <tr>
    </tr>
    <tr>
        <td align="center">0</td>
        <td align="center">4</td>
        <td align="center">2</td>
        <td align="center">6</td>
        <td align="center">1</td>
        <td align="center">5</td>
        <td align="center">3</td>
        <td align="center">7</td>
    </tr>
</table>


可以发现规律：原序列的每个元素的下标用二进制表示，将二进制为翻转之后，得到的数就是拆分到最后所在的位置。如上图中 $1$ 是 `001` ，$4$ 是 `100` ，二进制恰好翻转，而在原序列与最终序列中，$1$ 和 $4$ 的位置恰好互换。于是考虑如何求出下标 $i$ 对应的互换位置 `rev[i]` 。
如果我们要将 $i$ 的二进制位翻转，我们可以将其拆成两部分，最后一位和剩下的所有位（称之为剩余位）。先将剩余位翻转，然后将最后一位放在翻转后的剩余位的前面，剩余位翻转的结果为：`rev[i >> 1] >> 1` ，再看情况在翻转后的剩余位之前加 $1$ 即可。

??? note "非递归 FFT"
    ```cpp
    #include <cstdio>
    #include <cmath>
    #include <complex>
    #include <algorithm>

    typedef std::complex<double> comp;

    const double pi = acos(-1.0);
    const int MAXN = 1e6 + 5;

    int n, m, k;
    int rev[MAXN << 2];
    comp F[MAXN << 2], G[MAXN << 2];

    void FFT(int lmt, comp arr[], int opt) {
        for (int i = 0; i < lmt; ++i) {
            if (rev[i] > i) std::swap(arr[i], arr[rev[i]]);
        }
        for (int len  = 2; len <= lmt; len <<= 1) {
            comp step(cos(2 * pi / len), opt * sin(2 * pi / len));
            for (int w = 0; w < lmt; w += len) {
                comp cur(1, 0);
                for (int i = w; i < w + len / 2; ++i) {
                    comp t1 = arr[i];
                    comp t2 = cur * arr[i + len / 2];
                    arr[i] = t1 + t2;
                    arr[i + len / 2] = t1 - t2;
                    cur *= step;
                }
            }
        }
        if (opt == -1) {
            for (int i = 0; i < lmt; ++i) {
                arr[i].real(arr[i].real() / lmt);
            }
        }
    }

    int main() {
        scanf("%d%d", &n, &m);
        for (int i = 0; i <= n; ++i) {
            scanf("%d", &k);
            F[i].real(k);
        }
        for (int i = 0; i <= m; ++i) {
            scanf("%d", &k);
            G[i].real(k);
        }
        int lmt = 1;
        while (lmt <= n + m) {
            lmt <<= 1;
        }
        for (int i = 1; i < lmt; ++i) {
            rev[i] = (rev[i >> 1] >> 1) | (i & 1 ? lmt >> 1 : 0);
        }
        FFT(lmt, F, 1);
        FFT(lmt, G, 1);
        for (int i = 0; i <= lmt; ++i) {
            F[i] *= G[i];
        }
        FFT(lmt, F, -1);
        for (int i = 0; i <= n + m; ++i) {
            printf("%.0lf ", fabs(F[i].real()));
        }
        return 0;
    }
    ```

#### FFT 与字符串匹配