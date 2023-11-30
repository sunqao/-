# 算法基础（三十七）：动态规划 - 树形DP

![image-20231129214724863](https://typora-1310242472.cos.ap-nanjing.myqcloud.com/typora_img/image-20231129214724863.png)

![image-20231129214743598](https://typora-1310242472.cos.ap-nanjing.myqcloud.com/typora_img/image-20231129214743598.png)

**状态表示：**

变量维度：树形DP，这里用两个变量`f[u, 0]`和`f[u, 1]`

变量对应集合：`f[u, 0]`对应的集合为所有从以`u`为根的子树中选择，不选`u`的方案的快乐指数构成的集合，`f[u, 1]`对应的集合为所有从以`u`为根的子树中选择，选`u`的方案的快乐指数构成的集合

变量属性：集合的最大值

**状态计算与集合划分：**

我们以下图为例来说明：

![image-20231129215316542](https://typora-1310242472.cos.ap-nanjing.myqcloud.com/typora_img/image-20231129215316542.png)

在计算`f[u, 0]`的时候，这时的方案肯定要两个子树方案都最大并且加起来，因此`f[u, 0] = Max(f[s1, 0], f[s1, 1]) + Max(f[s2, 0], f[s2, 1]) `

同理，`f[u, 1]`表示选择了点`u`，因为不会有下属愿意和直接上司一起参加，所以`f[u, 1] = f[s1, 0] + f[s2, 0] + a[u]`，`a[u]`表示点`u`的快乐值

这样最后的结果就是`Max(f[root, 0], f[root, 1])`

当然每个结点不一定只有两个儿子，所以将公式统一就是：

$f[u, 0] = \Sigma Max(f[si, 0], f[si, 1])$

$f[u, 1] = \Sigma Max(f[si, 0])$

**时间复杂度：**

一个有`n`个节点，我们每个点需要枚举两次，每次需要枚举它的所有的儿子，所有节点的儿子数量是`n - 1`，因此一共枚举了`2(n - 1)`次，时间复杂度是`O(n)`

**代码实现：**

```cpp
 #include<iostream>
#include<algorithm>
#include<cstring>
using namespace std;

const int N = 6010;

int n;
int happy[N];
int h[N], e[N], ne[N], idx;//邻接表来存储图

//idx, 空闲结点的地址
//e[idx], 结点的元素
//ne[idx], 结点指向下一个元素的地址
//h[i]元素i链表的头指针


int f[N][2];

bool has_father[N];//一个布尔数组，判断每个点是否有父节点

void add(int a, int b){
    e[idx] = b;//分配一个结点存储元素b
    ne[idx] = h[a];//这个元素插入到a结点链表的头部，这个节点指向当前头结点指向的位置
    h[a] = idx;//结点a的头指针指向当前存储b的地址
    idx ++;//空闲指针移动一位，为下一个结点分配空闲结点
    
}

void dfs(int u){
    //如果选了u这个结点，首先需要加上这个点的快乐值
    f[u][1] = happy[u];
    
    //枚举这个点的所有的儿子
    //也就是u的邻接表的所有值
    for(int i = h[u]; i != -1; i = ne[i]){
        int j = e[i];//j是u的每个儿子
        dfs(j);//先算j, 假设算出了f[j][0]与f[j][1]
        f[u][0] += max(f[j][0], f[j][1]);//加上它的两个儿子的方案
        f[u][1] += f[j][0];//加上不选当前这个儿子得到的最大值
    }
}

int main(){
    scanf("%d", &n);
    //先读入所有点的高兴度
    for(int i = 1; i <= n; i ++) scanf("%d", &happy[i]);
    
    //初始化邻接表的表头
    memset(h, -1, sizeof h);
    
    //读入每条边，一共有n - 1条边
    for(int i = 0; i < n - 1; i ++){
        int a, b;
        //b是a的父节点
        scanf("%d%d", &a, &b);
        //a有父节点了
        has_father[a] = true;
        //在邻接表中插入一条边b -> a
        add(b, a);
        
    }
    
    int root = 1;//从1开始枚举, 找到树的根结点
    while(has_father[root]) root ++;
    
    dfs(root);
    
    printf("%d\n", max(f[root][0], f[root][1]));
    
    return 0;
}
```

