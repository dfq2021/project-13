# project-13
Implement the above ECMH scheme

# 实验原理
ECMH通过把哈希映射成椭圆曲线上的点，然后利用ECC进行运算，可以把多个数据的 hash 合并到一个 hash 中，并且支持删除。这样节点维护一个 UTXO 的根 hash 的成本就很低了，每次只需做增量修改。然后只需要把 UTXO 根 hash 记录到区块上，其他节点同步 UTXO 集合之后，就可以验证该集合是否被篡改了。该方案的缺点是只能做全量验证，没办法验证单独一个 UTXO 是否存在。
# 实现方式
### 1.二次剩余的实现
我们使用勒让德符号来判断、求取二次剩余
```python
def tonelli(n, p):  
    # 勒让德符号
    def legendre(a, p):
        return pow(a, (p - 1) // 2, p)

    if (legendre(n, p) != 1):
        return -1
    q = p - 1
    s = 0
    while q % 2 == 0:
        q //= 2
        s += 1
    if s == 1:
        return pow(n, (p + 1) // 4, p)
    for z in range(2, p):
        if p - 1 == legendre(z, p):
            break
    c = pow(z, q, p)
    r = pow(n, (q + 1) // 2, p)
    t = pow(n, q, p)
    m = s
    t2 = 0
    while (t - 1) % p != 0:
        t2 = (t * t) % p
        for i in range(1, m):
            if (t2 - 1) % p == 0:
                break
            t2 = (t2 * t2) % p
        b = pow(c, 1 << (m - i - 1), p)
        r = (r * b) % p
        c = (b * b) % p
        t = (t * c) % p
        m = i
    return r
```
### 2.ECMH

（1）ECMH我们不停地将消息更新为它的哈希值，倘若此哈希值在椭圆曲线上有解，则通过二次剩余将其求出， 并输出此节点。如此我们即找到了一个正确的解。
```python
def ECMH(M):
    global a, b, p
    while (1):
        M = hash(M)
        tmp = M ** 3 + a * M + b
        tmp = tmp % p
        y = tonelli(tmp, p)
        if (y == -1):
            continue
        return tmp, y

```
（2）ECMH_Group函数的实现们需要将所有元素单独求解得到一堆点，之后将其相加即可。
```python
def ECMH_Group(M_Set):
    global a, b, p
    H = []
    h = 0
    for M in M_Set:
        while (1):
            M = hash(M)
            tmp = M ** 3 + a * M + b
            tmp = tmp % p
            y = tonelli(tmp, p)
            if (y == -1):
                continue
            H.append([tmp, y])
            h = h + 1
            break
    Hash = H[0]
    for i in range(1, h):
        Hash = T_add(Hash, H[i])
    return Hash[0], Hash[1]

```
# 实验结果
![image](https://github.com/jlwdfq/project-13/assets/129512207/bd7dd3b1-f293-4836-a0eb-de57a8fb2705)

运行时间：0.0010013580322265625 s
# 实验环境
| 语言  | 系统      | 平台   | 处理器                     |
|-------|-----------|--------|----------------------------|
| Cpp   | Windows10 | pycharm| Intel(R) Core(TM)i7-11800H |
# 小组分工
戴方奇 202100460092 单人组完成project13

