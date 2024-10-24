![image-20241022212115244](images/image-20241022212115244.png)

# 准备工作

## 显示立方体的uv

![image-20241022214222662](images/image-20241022214222662.png)

u越大的方向r值越大，v越大的方向g值越大，因此导致了这种结果：

<img src="images/image-20241022214540873.png" alt="image-20241022214540873" style="zoom:50%;" />

## 将立方体的uv的原点置于面的中心

![image-20241022215459538](images/image-20241022215459538.png)

<img src="images/image-20241022215511189.png" alt="image-20241022215511189" style="zoom:50%;" />



# SDF（有向距离场）

以下代码创造了一个半径为0.5的球形的SDF

![image-20241023133325890](images/image-20241023133325890.png)

![image-20241023134209429](images/image-20241023134209429.png)

<img src="images/image-20241023134222523.png" alt="image-20241023134222523" style="zoom:50%;" />

修改该值[-0.5，0.5]

![image-20241023134321937](images/image-20241023134321937.png)

可以观测到球形的SDF在二维平面上穿梭

<img src="images/image-20241023134406422.png" alt="image-20241023134406422" style="zoom:50%;" />

## step函数

step函数可以让以上特征更为明显

<img src="images/image-20241023133005215.png" alt="image-20241023133005215" style="zoom:50%;" />

红色部分值为1，黑色部分值为0

<img src="images/image-20241023134639582.png" alt="image-20241023134639582" style="zoom:50%;" />



## SDF的性质

- 并

<img src="images/image-20241023135832379.png" alt="image-20241023135832379" style="zoom:50%;" />



- 交

<img src="images/image-20241023135752336.png" alt="image-20241023135752336" style="zoom:50%;" />

- 差

<img src="images/image-20241023135907486.png" alt="image-20241023135907486" style="zoom:50%;" />

## 渐变

![image-20241023140210926](images/image-20241023140210926.png)

### 应用场景

#### 矢量字体

![image-20241023140719894](images/image-20241023140719894.png)

#### 描边

![image-20241023143207292](images/image-20241023143207292.png)

# Raymarching 求交

![image-20241023145836643](images/image-20241023145836643.png)

![image-20241023153604030](images/image-20241023153604030.png)

## 定义GetDist (获取SDF距离)

在（0，0，0）点有个半径大小为0.5的sphere SDF

```c
float GetDist(float3 p)
{
    // 在（0，0，0）点有个半径大小为0.5的sphere
    float d = length(p)-0.5;
    return d;
}
```

### 光线步进求交

```c
float Raymarch(float3 ro,float3 rd)
{
    float dO = 0;
    float dS;
    for (int i=0;i<MAX_STEPS;i++)
    {
        float3 p = ro + dO * rd; // 光线在dO时刻所在的点p
        dS = GetDist(p); // 计算p到球表面的距离
        dO += dS;
        if (dS<SURF_DIST || dO>MAX_DIST)break;
    }
    return dO;
}
```

```c
fixed4 frag(v2f i) : SV_Target
{
    float2 uv = i.uv-0.5; // 将uv的原点移到面的中心
    float3 ro = float3(0, 0, -3); // 设置虚拟相机位置
    float3 rd = normalize(float3(uv.x, uv.y, 1)); // 光线方向

    float d = Raymarch(ro, rd);

    fixed4 col = 0;
    // 如果d < MAX_DIST，说明击中了表面，设置为红色
    if (d < MAX_DIST)
    {
        col.r = 1;
    }
    return col;
}
```

<img src="images/image-20241023212025349.png" alt="image-20241023212025349" style="zoom:33%;" />

# 求球表面法线

```c
// 求得球表面法线
float3 GetNormal(float3 p)
{
    float2 e = float2(1e-2, 0); // 微小的距离
    float3 n = GetDist(p) - float3(
        GetDist(p-e.xyy),
        GetDist(p-e.yxy),
        GetDist(p-e.yyx)
    );
    return normalize(n);
}
```

<img src="images/image-20241024110220433.png" alt="image-20241024110220433" style="zoom:50%;" />

# 环形距离场

```c
float GetDist(float3 p)
{
    // 在（0，0，0）点有个半径大小为0.5的sphere
    float d = length(p)-0.5;
    d = length(float2(length(p.xy) - 0.5, p.z)) - 0.1;
    return d;
}
```



# 精华

float3 p = ro + dO * rd; // 光线在dO时刻所在的点p

这个公式可以用在别的行进行为上

<img src="images/image-20241022221256581.png" alt="image-20241022221256581" style="zoom:50%;" />