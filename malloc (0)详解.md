问题： 

```
char* ptr = (char*)malloc(0 * sizeof(char));

if (NULL == ptr)
	printf("got a NULL pointer");
else
	printf("got a Valid pointer");
```

请问：上面的程序输出为什么？在C99的标准里面解释到，如果给malloc传递0参数，其返回值是依赖于编译器的实现，但是不管返回何值，该指针指向的对象是不可以访问的。在VC6编译环境下，输出“got a Valid pointer”

但是我试图给该指针赋值，如：ptr = ''a'' ; 编译器并没有给出任何错误和警告信息，接着，我再输出该值，printf("ptr=%d\n", *ptr) ;也可以正常输出。

但是当我用free(ptr) ; 释放内存的时候，出现错误，为什么呢？下面是我看了网友经过讨论以后我比较认同的看法：

当malloc分配内存时它除了分配我们指定SIZE的内存块，还会分配额外的内存来存储我们的内存块信息，用于维护该内存块。因此，malloc（0）返回一个合法的指针并指向存储内存块信息的额外内存，我们当然可以在该内存上进行读写操作，但是这样做了会破坏该内存块的维护信息，因此当我们调用free(ptr)时就会出现错误。完整程序如下：

```
#include <iostream>
using namespace std;

int main()
{
	char *ptr;
	ptr = (char*)malloc(0 * sizeof(char));

	if (NULL == ptr)
		printf("got a NULL pointer\n");
	else
	{
		printf("got a Valid pointer\n");
		ptr = "a";
		printf("the value at %X is:%c\n", ptr, *ptr);
		free(ptr);//if we did not add this statement ,the program can run normnlly, 
	}
	return 0;
}
```

既然malloc另外分配内存来维护该内存块，也就是说分配来用于维护该内存块的内存的大小也是有限的，那么到底是多少呢？这和可能也依赖于实现，在VC6下，是56BYTE,下面是测试程序： 

```
#include <iostream>
using namespace std;

int main()
{
	char *ptr;
	ptr = (char*)malloc(0 * sizeof(char));

	if (NULL == ptr)
		printf("got a NULL pointer\n");
	else
	{
		printf("got a Valid pointer\n");
		strcpy(ptr, "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa");
		//printf("the value at %X is:%c\n",ptr,*ptr);
		printf("the string at %x is :%s\n", ptr, ptr);
		//free(ptr);//if we did not add this statement ,the program can run normnlly, 
	}
	return 0;
}
```

此时我们没有把free（ptr）编译进来，同样会发生异常，程序输出很多个56个a,我暂时还不明白为什么？？？？？如果把free(ptr); 编译进来，就会发生运行错误！

通过上面的讨论和程序的验证，确实证明了网友和我的想法是正确的，也就是malloc(0)还会额外分配一部分空间（在VC6下是56字节）用于维护内存块。