这题让我们知道了Java中int整数的边界判断问题

int的边界  `-2^31 ~ 2^31 - 1`

对于我们常规的思路，对边界的判断：

```Java
int num = c-'0';
if(res > Integer.MAX_VALUE/10 || (res == Integer.MAX_VALUE/10&&num>Integer.MAX_VALUE%10)){
    return Integer.MAX_VALUE;
}

if(res < Integer.MIN_VALUE/10 || (res == Integer.MIN_VALUE/10&&-num<Integer.MIN_VALUE%10)){
    return Integer.MIN_VALUE;
}
res = res*10+sign*num;
```

对于Integer.parseInt方法中对String转int的实现，他一次便完成了对正数和负数的越界判断！！

```Java
            multmin = limit / 10;
            while (i < len) {
                digit = Character.digit(s.charAt(i++), 10);
                // 如果不是数字，是字母或者其他的
                if (digit < 0) {
                    return negative ? result : -result;
                }
                /*下面这两个判断是跟
                if(res < Integer.MIN_VALUE/10 || (res == Integer.MIN_VALUE/10&&-num<Integer.MIN_VALUE%10)){
                    return Integer.MIN_VALUE;
                }
                if(res > Integer.MAX_VALUE/10 || (res == Integer.MAX_VALUE/10&&num>Integer.MAX_VALUE%10)){
                    return Integer.MAX_VALUE;
                }
                而且由于他只计算负数，因此就不用判断正数的最大值，只需要判断负数的最小值
                很巧妙，它将正数最大值，和负数最小值放在一起判断！！！
                 */
                // 那么这一步实在比较十位的大小
                if (result < multmin) {
                    return negative ? Integer.MIN_VALUE : Integer.MAX_VALUE;
                }
                result *= 10;
                // 对于这个判断，我们要知道result是负数，limit是负数，digit是正数
                //这一步是在比较个位的大小
                if (result < limit + digit) {
                    return negative ? Integer.MIN_VALUE : Integer.MAX_VALUE;
                }
                result -= digit;
            }
        } else {
            return 0;
        }
```

