---
layout:	post
title: "Hash"
subtitle: "Not done yet"
date: 2021-03-16 18:00:00
author: "Yinwhe"
header-style: text
tags:
    - Cryptology
---



# MD5

`MD` 即 `Message Digest` 报文摘要，是一种单向函数， 没有逆函数

## `Function`

**特点** - 输入任意长度的信息，经过处理，输出为`128`位的信息，即摘要，也称数字指纹；不同的输入得到的不同的结果，具有唯一性；

**用处** - 防止篡改、密码管理、数字签名

## `Principle`

### 分块

MD5是分块计算的，每个块的大小为**`64字节`**，当最后一块大小不足64字节时，要按以下步骤补充数据凑成64字节:

1. 假定该块大小`n<56`字节, 则在末尾补上以下数据: `0x80 0x00 0x00 0x00 ... 0x00`，共56-n个

    例如`n==55`时，只要补0x80一个字节;

    当`n==54`时，要补上0x80及0x00两个字节

2. 假定该块大小`n在[56,63]`范围内时, 则应在末尾补上`64-n+56`个字节。

    例如当`n==56`时, 应该补上64个字节;

    当`n==57`时, 应该补上63个字节

3. 最后补上`8`个字节，这8个字节相当于一个64位的整数, 它的值=message总共的**`位数`**(不含填充内容)

```
比如有一个文件的内容为'A'，仅一个字节，则:
A, 0x80,0x00,…,0x00,  0x08,0x00,0x00,…,0x00
   ~~~~~~~~~~~~~~~~   =====================
		共补上55个字节      共8字节, 其类型为64位整数
                      =0x00 00 00 00 00 00 00 08 (即8个bits)
```

### Vital Structure

```cpp
typedef struct _MD5_CTX
{
   unsigned long  state[4]; /* 128位摘要 */
   unsigned long  count[2]; /* 已处理的报文的二进制位数,最大值=2^64-1 */
   unsigned char  data[64]; /* 64字节message块 */
} MD5_CTX;
```

### Procedure

> Not fully understood, review occasionally.

主程序如下：

```cpp
int main(int argc, char* argv[])
{
   MD5_CTX m;
   unsigned char s[100];
   int i;
   unsigned char *p;
   puts("Input a message:");
   gets(s);
   Init_MD5(&m);
   Update_MD5(&m, s, strlen(s));
   Final_MD5(&m);
   p = (unsigned char *)m.state;
   puts("The 128-bit md5 digest is:");
   for(i=0; i<16; i++)
      printf("%02X ", p[i]);
   getchar();
   return 0;
}
```

首先是初始化 **`Init`** 

作用： count 归零，state设置初始值

```cpp
int Init_MD5(MD5_CTX *MD5_ctx)
{
   MD5_ctx->count[0] = 0;
   MD5_ctx->count[1] = 0;
   MD5_ctx->state[0] = 0x67452301; // a0
   MD5_ctx->state[1] = 0xEFCDAB89; // b0
   MD5_ctx->state[2] = 0x98BADCFE; // c0
   MD5_ctx->state[3] = 0x10325476; // d0
   return 0;
}
```

- 这里的state也称为幻数；标准的幻数是 `A=(01234567)16，B=(89ABCDEF)16，C=(FEDCBA98)16，D=(76543210)16`；如果在程序中定义应该是: `A=0X67452301L，B=0XEFCDAB89L，C=0X98BADCFEL，D=0X10325476L`

然后进入四轮循环计算 (也即 `**Update**` ) 循环的次数是分组个数+1，因为第一次是没有加入分组的

作用： 增加count值即报文位数，分块（64字节）计算并更新到state，多余的存在data里等待之后的填充

```cpp
int Update_MD5(MD5_CTX *MD5_ctx, unsigned char *buf, unsigned long buf_len){
   unsigned long i, bytes_left, bytes_to_fill;
   bytes_left = (MD5_ctx->count[0] >> 3) & 0x3F; // 内部缓冲data中剩余未处理的字节数
																								 // >>3即 /8，将bit转换为byte, &0x3F 等价于 %64
   
   *(unsigned __int64 *)MD5_ctx->count += (unsigned __int64)buf_len << 3; // 更新报文位数
   bytes_to_fill = 64 - bytes_left; // data中需要补充的字节数
   if (buf_len >= bytes_to_fill) {
      memcpy(&MD5_ctx->data[bytes_left], buf, bytes_to_fill); // 现在data中已充满64字节
      Process_One_Block_MD5 (MD5_ctx->state, MD5_ctx->data); // 对data中的64字节进行计算
      bytes_left = 0; // data中剩余未处理字节数变0
      for (i=bytes_to_fill; i+63 < buf_len && i+63 >= 63; i+=64){ // 从buf复制64字节到data
         // i=bytes_to_fill是为了跳过buf中已经补充给data的bytes_to_fill个字节的数据
         // i+63 < buf_len是为了保证buf中剩余数据至少还有64字节
         // i+63 >= 63是为了防止i+63发生溢出(注意i是unsigned long, 当它接近2^32-1时, 
         // i+63会溢出), 若i+63<63即i+63溢出, 则意味着buf中剩余字节不足64字节
         Process_One_Block_MD5(MD5_ctx->state, &buf[i]);
      }
   }
   else i = 0;

   memcpy(&MD5_ctx->data[bytes_left], &buf[i], buf_len-i);
   return 0;
}

static void Process_One_Block_MD5(unsigned long state[4], unsigned char block[64])
{
   unsigned long a = state[0], b = state[1], c = state[2], d = state[3];
   unsigned long *px = (unsigned long *)block;
	 //Round 1
   FF(a, b, c, d, px[0],  7,  0xD76AA478);   // step 1: a1
   FF(d, a, b, c, px[1],  12, 0xE8C7B756);   // step 2: d1
   FF(c, d, a, b, px[2],  17, 0x242070DB);   // step 3: c1
   FF(b, c, d, a, px[3],  22, 0xC1BDCEEE);   // step 4: b1
   FF(a, b, c, d, px[4],  7,  0xF57C0FAF);   // step 5: a2
   FF(d, a, b, c, px[5],  12, 0x4787C62A);   // step 6: d2
   FF(c, d, a, b, px[6],  17, 0xA8304613);   // step 7: c2
   FF(b, c, d, a, px[7],  22, 0xFD469501);   // step 8: b2
   FF(a, b, c, d, px[8],  7,  0x698098D8);   // step 9: a3
   FF(d, a, b, c, px[9],  12, 0x8B44F7AF);   // step 10:d3
   FF(c, d, a, b, px[10], 17, 0xFFFF5BB1);   // step 11:c3
   FF(b, c, d, a, px[11], 22, 0x895CD7BE);   // step 12:b3
   FF(a, b, c, d, px[12], 7,  0x6B901122);   // step 13:a4
   FF(d, a, b, c, px[13], 12, 0xFD987193);   // step 14:d4
   FF(c, d, a, b, px[14], 17, 0xA679438E);   // step 15:c4
   FF(b, c, d, a, px[15], 22, 0x49B40821);   // step 16:b4
   //Round 2                                
   GG(a, b, c, d, px[1],  5,  0xF61E2562);   // step 17:a5
   GG(d, a, b, c, px[6],  9,  0xC040B340);   // step 18:d5
   GG(c, d, a, b, px[11], 14, 0x265E5A51);   // step 19:c5
   GG(b, c, d, a, px[0],  20, 0xE9B6C7AA);   // step 20:b5
   GG(a, b, c, d, px[5],  5,  0xD62F105D);   // step 21:a6
   GG(d, a, b, c, px[10], 9,  0x2441453);    // step 22:d6
   GG(c, d, a, b, px[15], 14, 0xD8A1E681);   // step 23:c6
   GG(b, c, d, a, px[4],  20, 0xE7D3FBC8);   // step 24:b6
   GG(a, b, c, d, px[9],  5,  0x21E1CDE6);   // step 25:a7
   GG(d, a, b, c, px[14], 9,  0xC33707D6);   // step 26:d7
   GG(c, d, a, b, px[3],  14, 0xF4D50D87);   // step 27:c7
   GG(b, c, d, a, px[8],  20, 0x455A14ED);   // step 28:b7
   GG(a, b, c, d, px[13], 5,  0xA9E3E905);   // step 29:a8
   GG(d, a, b, c, px[2],  9,  0xFCEFA3F8);   // step 30:d8
   GG(c, d, a, b, px[7],  14, 0x676F02D9);   // step 31:c8
   GG(b, c, d, a, px[12], 20, 0x8D2A4C8A);   // step 32:b8
   //Round 3
   HH(a, b, c, d, px[5],  4,  0xFFFA3942);   // step 33:a9
   HH(d, a, b, c, px[8],  11, 0x8771F681);   // step 34:d9
   HH(c, d, a, b, px[11], 16, 0x6D9D6122);   // step 35:c9
   HH(b, c, d, a, px[14], 23, 0xFDE5380C);   // step 36:b9
   HH(a, b, c, d, px[1],  4,  0xA4BEEA44);   // step 37:a10
   HH(d, a, b, c, px[4],  11, 0x4BDECFA9);   // step 38:d10
   HH(c, d, a, b, px[7],  16, 0xF6BB4B60);   // step 39:c10
   HH(b, c, d, a, px[10], 23, 0xBEBFBC70);   // step 40:b10
   HH(a, b, c, d, px[13], 4,  0x289B7EC6);   // step 41:a11
   HH(d, a, b, c, px[0],  11, 0xEAA127FA);   // step 42:d11
   HH(c, d, a, b, px[3],  16, 0xD4EF3085);   // step 43:c11
   HH(b, c, d, a, px[6],  23, 0x04881D05);   // step 44:b11
   HH(a, b, c, d, px[9],  4,  0xD9D4D039);   // step 45:a12
   HH(d, a, b, c, px[12], 11, 0xE6DB99E5);   // step 46:d12
   HH(c, d, a, b, px[15], 16, 0x1FA27CF8);   // step 47:c12
   HH(b, c, d, a, px[2],  23, 0xC4AC5665);   // step 48:b12
   //Round 4
   II(a, b, c, d, px[0],  6,  0xF4292244);   // step 49:a13
   II(d, a, b, c, px[7],  10, 0x432AFF97);   // step 50:d13
   II(c, d, a, b, px[14], 15, 0xAB9423A7);   // step 51:c13
   II(b, c, d, a, px[5],  21, 0xFC93A039);   // step 52:b13
   II(a, b, c, d, px[12], 6,  0x655B59C3);   // step 53:a14
   II(d, a, b, c, px[3],  10, 0x8F0CCC92);   // step 54:d14
   II(c, d, a, b, px[10], 15, 0xFFEFF47D);   // step 55:c14
   II(b, c, d, a, px[1],  21, 0x85845DD1);   // step 56:b14
   II(a, b, c, d, px[8],  6,  0x6FA87E4F);   // step 57:a15
   II(d, a, b, c, px[15], 10, 0xFE2CE6E0);   // step 58:d15
   II(c, d, a, b, px[6],  15, 0xA3014314);   // step 59:c15
   II(b, c, d, a, px[13], 21, 0x4E0811A1);   // step 60:b15
   II(a, b, c, d, px[4],  6,  0xF7537E82);   // step 61:a16
   II(d, a, b, c, px[11], 10, 0xBD3AF235);   // step 62:d16
   II(c, d, a, b, px[2],  15, 0x2AD7D2BB);   // step 63:c16
   II(b, c, d, a, px[9],  21, 0xEB86D391);   // step 64:b16

   state[0] += a;
   state[1] += b;
   state[2] += c;
   state[3] += d;
}
```

- 需要注意的是，每次state都是要更新的，与计算的结果相加。
- 这里用到的函数如下：

    ```cpp
    #define F(x,y,z) ((x & y) | (~x & z))
    #define G(x,y,z) ((x & z) | (y & ~z))
    #define H(x,y,z) (x^y^z)
    #define I(x,y,z) (y ^ (x | ~z))
    #define ROTATE_LEFT(x,n) ((x << n) | (x >> (32-n)))
    #define FF(a,b,c,d,x,s,ac) \
              { \
              a += F(b,c,d) + x + ac; \
              a = ROTATE_LEFT(a,s); \
              a += b; \
              }
    #define GG(a,b,c,d,x,s,ac) \
              { \
              a += G(b,c,d) + x + ac; \
              a = ROTATE_LEFT(a,s); \
              a += b; \
              }
    #define HH(a,b,c,d,x,s,ac) \
              { \
              a += H(b,c,d) + x + ac; \
              a = ROTATE_LEFT(a,s); \
              a += b; \
              }
    #define II(a,b,c,d,x,s,ac) \
              { \
              a += I(b,c,d) + x + ac; \
              a = ROTATE_LEFT(a,s); \
              a += b; \
              }
    ```

然后就是对结尾的处理**`Final`**

作用： 为最后一块填充若小于56字节则补充道56，若大于补充完当前块并补充道下一个56字节，最后增加8字节报文长度

```cpp
int Final_MD5(MD5_CTX *MD5_ctx)
{
   unsigned long bytes_left, pad_len;
   unsigned char total_bits[8];
   memcpy(total_bits, (unsigned char *)MD5_ctx->count, 8); // total_bits=已处理的报文的二进制位数
                                                           // (含data中剩余的字节)
                                                           // 后面补充的pad_stuff及
                                                           // total_bits本身不计在内
   bytes_left = (MD5_ctx->count[0] >> 3) & 0x3F;
   pad_len = (bytes_left < 56) ? (56 - bytes_left) : 
               (64 - bytes_left + 56); // bytes_left==56时, 要补8+56=64字节
                                       // bytes_left==57时, 要补7+56=63字节
   Update_MD5(MD5_ctx, pad_stuff, pad_len); // 把pad_stuff加到data中计算
   Update_MD5(MD5_ctx, total_bits, 8); // 把total_bits加到data中计算
   return 0;
}
```

- 其中pad_stuff是填充物，定义为

    ```cpp
    static unsigned char pad_stuff[64] = {
       0x80,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
          0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
          0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
          0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0
    };
    ```

## 碰撞

> @饭饭 提供

所谓碰撞，即当`m1≠m2`时，有`md5(m1) == md5(m2)` ，其危害是巨大的。

一个破解方法是**彩虹表**，也即预计算破解，其思路为**枚举**存放与密码对应的MD5，以8位密码为例：

1. 8位大小写字母与数字，每位有62种变化，共有 $62^8$ 种结果

2. 每种变化需要 $8+16=24$ 字节存储

3. 存储空间为 $62^8\times 24/2^{40}=4700TB$

   需要的存储空间太大，使用`rainbow table`的算法用时间换空间

4. rainbow table（编程体验加深理解）

   - 建立4位字符串与数字的映射

   - 获取数据库中的初始数据

     ```c
     for(i=1; i<=SomeBigNumber; i++)
     {
       ①产生一个随机数n0, 其中n0∈[0,26^4-1]
       找出与n0对应的字母组合p0, 计算m0=md5(p0)
       for(j=1; j<=100; j++)
       {
         ②计算nj = mj-1 mod 26^4/* 消减函数 */
         ③以nj为序号,找出与它对应的4个英文字母组合pj。
         例如n=0时p="AAAA"，n=1时p="AAAB"，n=25时
         p="AAAZ", n=26时p="AABA"。
         ④计算mj = md5(pj)
       }
       ⑤在数据库中记录步骤①所得的n0及j==100时步骤④所得的m100。
     }
     ```

     最后只保存`n0`, `m100`到数据库中，**并对** `m100` **做排序**

   - 破解某个md5的值`m`是由哪4个字母算出来的，则将`m`与数据库中的md5值进行比较：

     情况1：成功匹配，从`n0`开始进行99次运算，求出`m99`，就能算出`n100 = m99 % 26` 再验证

     情况2：若无法匹配，则根据`m`计算新的`n'`，然后进一步计算出新的 `m'`，再与数据库中存储的数据比较，若成功匹配则返回情况1进行计算，若失败则重复情况2

   - 数据量大时**不能完全覆盖，也不能完全避免碰撞**

# SHA1

**sha-1散列算法**计算出来的hash值有`160`位，即`20`字节，比md5多了32位。

sha-1也是**分块**计算，每块也是64字节，当最后一块不足64字节也按照md5的方式进行填充；数据块的最后一定要补上表示报文总共位数的8个字节。

一些细节：

1. SHA1计算时每次处理16个ulong，但是会将其扩充为80个ulong，然后每20个进行计算
2. SHA1根据大端设计，小端需要定义一个 `LITTLE_ENDIAN` 进行转换