## 题目

统计所有小于非负整数 *`n`* 的质数的数量。

## 解答

#### 暴力枚举

~~~java
class Solution {
    public int countPrimes(int n) {
        int ans = 0;
        for (int i = 2; i < n; ++i) {
            ans += isPrime(i) ? 1 : 0;
        }
        return ans;
    }

    public boolean isPrime(int x) {
        for (int i = 2; i * i <= x; ++i) {
            if (x % i == 0) {
                return false;
            }
        }
        return true;
    }
}
~~~

#### 厄拉多塞筛法

我们考虑这样一个事实：如果 x 是质数，那么大于 x 的 x 的倍数 2x,3x,… 一定不是质数，因此我们可以从这里入手。

我们设 isPrime[i] 表示数 i是不是质数，如果是质数则为1，否则为0。从小到大遍历每个数，如果这个数为质数，则将其所有的倍数都标记为合数（除了该质数本身），即0，这样在运行结束的时候我们即能知道质数的个数。

这种方法的正确性是比较显然的：这种方法显然不会将质数标记成合数；另一方面，当从小到大遍历到数x时，倘若它是合数，则它一定是某个小于x 的质数y的整数倍，故根据此方法的步骤，我们在遍历到y时，就一定会在此时将x标记为isPrime[x]=0。因此，这种方法也不会将合数标记为质数。

当然这里还可以继续优化，对于一个质数 x，如果按上文说的我们从 2x 开始标记其实是冗余的，应该直接**从x*x 开始标记**，因为2x,3x,… 这些数一定在x之前就被其他数的倍数标记过了，例如2的所有倍数，3的所有倍数等。

~~~java
class Solution {
    public int countPrimes(int n) {
        int[] isPrime = new int[n];
        Arrays.fill(isPrime, 1);
        int ans = 0;
        for (int i = 2; i < n; ++i) {
            if (isPrime[i] == 1) {
                ans += 1;
                if ((long) i * i < n) {
                    for (int j = i * i; j < n; j += i) {
                        isPrime[j] = 0;
                    }
                }
            }
        }
        return ans;
    }
}
~~~

