- 顺序查找设置哨兵

```
//数组r[1] ~ r[n]存放查找集合
int SeqSearch(int r[],int n ,ing k){
    int i = n;
    r[0] = k; //设置哨兵,
    while(r[i] != k) //不用检测比较位置是否越界
        i--;
    return i;
}
```
