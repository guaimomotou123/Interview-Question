## memmove 和 memcpy的区别以及处理内存重叠问题

区别：

memcpy和memmove（）都是C语言中的库函数，在头文件string.h中，作用是拷贝一定长度的内存的内容，原型分别如下：

```
void *memcpy(void *dst, const void *src, size_t count);
void *memmove(void *dst, const void *src, size_t count);
```

他们的作用是一样的，唯一的区别是，当内存发生局部重叠的时候，memmove保证拷贝的结果是正确的，memcpy不保证拷贝的结果的正确。

 

一、memcpy函数

Memcpy原型：   

```
void *memcpy(void *dest, const void *src, size_t n);
```

描述：

​        memcpy()函数从src内存中拷贝n个字节到dest内存区域，但是源和目的的内存区域不能重叠。

返回值：

​        memcpy()函数返回指向dest的指针。

 

二、memmove函数

memmove原型：

```
void *memmove(void *dest, const void *src, size_t n);
```

描述：        

memmove() 函数从src内存中拷贝n个字节到dest内存区域，但是源和目的的内存可以重叠。 

返回值：         

memmove函数返回一个指向dest的指针。  从上面的描述中可以看出两者的唯一区别就是在对待重叠区域的时候，memmove可以正确的完成对应的拷贝，而memcpy不能。  

内存覆盖的情形有以下两种: 

![](F:\F\Interview Question\image\7.png)

先看memcpy()和memmove()这两个函数的实现： 

```
void* my_memcpy(void* dst, const void* src, size_t n)
{
    char *tmp = (char*)dst;
    char *s_src = (char*)src;
 
    while(n--) {
        *tmp++ = *s_src++;
    }
    return dst;
}
```

从实现中可以看出memcpy()是从内存左侧一个字节一个字节地将src中的内容拷贝到dest的内存中，这种实现方式导致了对于图中第二种内存重叠情形下，最后两个字节的拷贝值明显不是原先的值了，新的值是变成了src的最开始的2个字节了。 

而对于第一种内存覆盖情况，memcpy的这种拷贝方式是可以的。

而memmove就是针对第二种内存覆盖情形，对memcpy进行了改进，改进代码如下：

```
void* my_memmove(void* dst, const void* src, size_t n)
{
    char* s_dst = (char*)dst;
    char* s_src = (char*)src;
    if(s_dst>s_src && (s_src+n>s_dst)) {      //-------------------------第二种内存覆盖的情形。
        s_dst = s_dst+n-1;
        s_src = s_src+n-1;
        while(n--) {
            *s_dst-- = *s_src--;
        }
    }else {
        while(n--) {
            *s_dst++ = *s_src++;
        }
    }
    return dst;
}
```

在第二种内存覆盖的情形下面，memcpy会出错，但是memmove是能正常工作的。 



此文章转载自：https://blog.csdn.net/li_ning_/article/details/51418400