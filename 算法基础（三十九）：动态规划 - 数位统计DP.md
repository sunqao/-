# 算法基础（三十九）：动态规划 - 数位统计DP

![image-20231106215302431](https://typora-1310242472.cos.ap-nanjing.myqcloud.com/typora_img/image-20231106215302431.png)

![image-20231106215315314](https://typora-1310242472.cos.ap-nanjing.myqcloud.com/typora_img/image-20231106215315314.png)

这道题目的核心思想在于分类讨论，要统计`a ~ b`中数字出现的次数，我们先统计`1 ~ b`中每个数字出现的次数，然后再减去`1 ~ a - 1`中数字出现的次数即可，那么来看`1 ~ num`中如何统计每个数字出现的次数，假设统计的是`x`出现的次数，且`num`是`abcdef`

我们要求从`00001 ~ abcdef`中`1`出现的次数，这个求法与求`1`在每一位出现的次数然后相加是完全等价的

假设我们求的是`x`在第三位上出现的次数，也就是`x`出现在`c`处的次数，此时`num = abcdef`，我们需要统计形如`ijxklm`这样的数在`000001 ~ abcdef`中有多少个，此时我们来分类讨论

**当`x > 0`的时候：**

1. 当`x`在中间位置时，形如`ijxklm`：
   1. 假设`ij < ab`，也就是`00 <= ij <= ab - 1`，此时`klm`可以取的数为`0 ~ 999`，因此第一类可取的总数为`ab * 1000`
   2. 假设`ij = ab`，即形如`abxkml`这样的数在`000001 ~ abcdef`中有多少个
      1. 如果`x > c`，可取的数为`0`
      2. 如果`x = c`，`klm`可取为`000 ~ def`，可取的数为`def + 1`
      3. 如果`x < c` , `klm`可取为`000 ~ 999`，可取的数为`1000`
   3. 假设`ij > ab`，可取的数为`0`
2. 当`x`在最高位时，形如`xijklm`
   1. 假设`x > a`，可取的数为`0`
   1. 假设`x = a`，`ijklm`可取的数为`00000 ~ bcdef`，一共`bcdef + 1`个
   1. 假设`x < a`，`ijklm`可取的数为`00000 ~ 99999`，一共`100000`个
3. 当`x`在最低位时，形如`ijklmx`
   1. 假设`ijklm < abcde`，此时`ijklm`可取的值为`00000 ~ abcde - 1`，因此第一类可取的总数为`abcde * 1`
   2. 假设`ijklm = abcde`
      1. 假如`x > f`，可取的数为`0`
      2. 假如`x = f`，可取的数为`1`
      3. 假如`x < f`，可取的数为`1`
   3. 假设`ijklm > abcde`，可取的数为`0`

然后将所有位的这些数的个数加起来即可，当前第三种情况可以合并到第一种情况中，将末尾的数当作`0`即可

**当`x = 0`的时候：**

当统计数字`0`的时候我们需要考虑前导零的问题，因为`000001`这样的数我们统计非零数字的时候是正常统计，但统计零的时候就不能带上前面的前导零，因为前面的前导零只是用来方便位数统计的，不能真实当作一个数的某一位，比如：

```cpp
0000001
0011100
```

这两个数中`1`出现了四次，零出现了两次，前导零不做统计

1. 当`x`在中间位置时，形如`ijxklm`：
   1. 假设`ij < ab`，也就是`01 <= ij <= ab - 1`，此时`klm`可以取的数为`0 ~ 999`，因此第一类可取的总数为`(ab - 1) * 1000`
   2. 假设`ij = ab`，即形如`abxkml`这样的数在`000001 ~ abcdef`中有多少个
      1. 如果`x > c`，可取的数为`0`，而且`x`也不可能大于`c`
      2. 如果`x = c`，`klm`可取为`000 ~ def`，可取的数为`def + 1`
      3. 如果`x < c` , `klm`可取为`000 ~ 999`，可取的数为`1000`
   3. 假设`ij > ab`，可取的数为`0`
2. 当`x`在最高位时，形如`xijklm`，可取的数为`0`，`0`不能出现在最高位，因为此时这个数被当作前导零，不做统计
3. 当`x`在最低位时，形如`ijklmx`
   1. 假设`ijklm < abcde`，此时`ijklm`可取的值为`00000 ~ abcde - 1`，因此第一类可取的总数为`abcde * 1`
   2. 假设`ijklm = abcde`
      1. 假如`x > f`，可取的数为`0`
      2. 假如`x = f`，可取的数为`1`
      3. 假如`x < f`，可取的数为`1`
   3. 假设`ijklm > abcde`，可取的数为`0`

可以看到，`x = 0`与`x > 0`讨论的区别在于两点：

1. 在中间位置的时候第一种情况，`x = 0`的时候需要减去`10^i`，`i`是`x`所在的位数
2. 在最高位的时候`x = 0`没有任何取值 

**代码实现：**

```cpp
#include<iostream>
#include<algorithm>
#include<vector>
using namespace std;

//求10^i
int power10(int x){
    int res = 1;
    
    while(x --) res *= 10;
    
    return res;
}

//abcdef求当x在d的时候abc是多少
//nums[0] = f
int get(vector<int> num, int l, int r){
    int res = 0;
    for(int i = l; i >= r; i --){
        res = res * 10 + num[i];
    }
    
    return res;
}


//从1 ~ n中统计x出现的次数
int count(int n, int x){
    
    //当n = 0的时候直接返回0
    if(!n) return 0;
    
    vector<int> num;//将n的每一位抠出来
    
    while(n){
        num.push_back(n % 10);
        
        n /= 10;
    }
    
    n = num.size();//让n = num的位数
    
    int res = 0;//答案
    //mmmxmm 与abcdef
    //从最高位开始枚举
    
    //这里将x处于最高位，最低位，中间位置合并起来了
    //同时合并了x > 0 和x = 0的情况
    //!x当x = 0的时候默认从次高位开始算了
    for(int i = n - 1 -!x; i >= 0; i --){
        //当mmm < abc的时候
        if(i < n - 1){//先算其他位的情况
            //get(n - 1, i + 1), 即当x在第i位的时候比如x在abcdef的d位，求n - 1位即a 到 i + 1位即c
            //也就是abc是多少
            
            //power10也就是x在d位时mmmxmm, mm可取的数的大小，10^i
            res += get(num, n - 1, i + 1) * power10(i);
            
            //如果x = 0，这时需要再减去10 ^ i
            if(!x) res -= power10(i);
        }
        
        //当mmm = abc的时候
            //d = x
            //mmmxmm, abcdef, 此时应该是def + 1个数
        if(num[i] == x){
            res += (get(num, i - 1, 0) + 1);
        }
        
            //当d > x
            //abcxmm, abcdef, 此时是100个数, 10^i
        if(num[i] > x) res += power10(i);
    }
    
    return res;
}

int main(){
    int a, b;
    
    while(cin >> a >> b, a || b){
        
        if(a > b) swap(a, b);
        
        //统计每一个数出现的次数
        for(int i = 0; i < 10; i ++){
            //统计1 ~ b中i出现的次数 - 1 ~ a - 1中i出现的次数
            cout << count(b, i) - count(a - 1, i) << ' ';
        }
        cout << endl;
    }
    
    return 0;
}
```

这里的代码实现很抽象，利用函数以及语言特性合并了所有的情况，我建议初学者一开始分类的时候分开来写，这样不容易出错，就目前的分类讨论来说数位统计类问题跟dp沾了一点点边，但不多=）
