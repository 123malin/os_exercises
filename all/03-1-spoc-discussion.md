# lec5 SPOC思考题


NOTICE
- 有"w3l1"标记的题是助教要提交到学堂在线上的。
- 有"w3l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

请简要分析最优匹配，最差匹配，最先匹配，buddy system 分配算法的优势和劣势，并尝试提出一种更有效的连续内存分配算法 (w3l1)
```
  + 采分点：说明四种算法的优点和缺点
  - 答案没有涉及如下3点；（0分）
  - 正确描述了二种分配算法的优势和劣势（1分）
  - 正确描述了四种分配算法的优势和劣势（2分）
  - 除上述两点外，进一步描述了一种更有效的分配算法（3分）
 ```

>答：最先匹配算法的优点是：1.简单，因为它是按照地址顺序排序的，所以每次释放空间的时候只需按照地址把空闲空间插进去插进去，并且看它前后是否有临近的空闲分区，如果有则把它合并到一起；2.每次都是顺序查找第一个比要分配区域大的区域分配，这样就会使得在高地址空间有大块的空闲分区。缺点是：1.容易产生较多的外部碎片；2分配大块时速度较慢，需要查找前面分剩下的大量碎片。
最佳匹配算法的优点在大部分分配块的尺寸较小时体现的比较好，表现为;1.可避免大的空闲分区被拆分；2.可减小外部碎片的大小；3.相对来说也比较简单。缺点是：1.能产生外部碎片；2.释放分区比较慢，也比较复杂；3.容易产生很多基本上没有用的小碎片。
最差匹配算法的优点在中等大小的分配较多时效果最好，因为这样可以避免出现太多的小碎片。缺点是：1.释放分区比较慢，也比较复杂；2.会产生外部碎片；3.容易破坏大的空闲分区，因此后续难以分配大的分区。
buddy system分配算法的优点是：1.处理内碎片问题较出色，可以把内碎片的大小限定在一定范围内；2.按2的幂次方大小进行分配内存块可以避免把把大的内存块拆的太碎，更重要的是使分配和释放过程迅速。缺点是:1.一个很小的块往往会阻碍一个大块的合并;2.如果所需内存块不是2的幂就会造成一定的浪费；3.拆分和合并涉及到较多的链表和位图操作，开销比较大。
我认为的更有效的连续内存分配算法：大体思路和最先匹配算法基本一致，但是需要对其缺点进行改进，首先可以设定一定的阈值来衡量一个块到底属于大块，中快和小块，当按地址顺序查找一个分配区时，如果该分配区的大小大于待分配区域且两者只差大于小块的阈值时进行分配，这样就可以避免出现太多的无用小块，当要分配一个较大的区域时（大于大块阈值）时，可以由后进行查找，即把空闲区域组织为一个双向链表，这样就可以避免大块分配时查找太慢的问题。

## 小组思考题

请参考ucore lab2代码，采用`struct pmm_manager` 根据你的`学号 mod 4`的结果值，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在应用程序层面(可以 用python,ruby,C++，C，LISP等高语言)来实现，给出你的设思路，并给出测试用例。 (spoc)

> 代码如下

```
#include <iostream>

using namespace std;

struct block
{
	int start;         //起始地址 
	int size;         //大小
	bool is_used;
	 block *prev,*next; 
};

block *first = new block;
void mm_init(int start, int size)
{
	first -> start = start;
	first -> size = size;
	first -> is_used = false;
	first -> prev = NULL;
	first -> next = NULL;
}

block* malloc(int size)
{
	block *p = first, *res = NULL;
	int free_max = -1;
	
	while(p!=NULL)
	{
		if((p->size > free_max) && (p->is_used == false))
		{
			free_max = p->size;
			res = p;
		}
		p = p -> next;
	}
	if(free_max < size)
	{
		cout << "Couldn't find a block that is bigger than size" << endl;
		return NULL;
	}
	block *remain = new block;
	remain->start = res->start + size;
	remain->size = res->size - size;
	remain->is_used = false;
	remain->prev = res;
	remain->next = res->next;
	res->size = size;
	res->next = remain;	
	res->is_used = true;
	return res;
}

void print_block()
{
	block *p = first;
	while(p!=NULL)
	{
		cout << "start location = " << p->start << " size = " << p->size <<" is_used = " << p->is_used << endl;
		p = p->next;
	}
}

void free(block *f)
{
	f->is_used = false;
	block *prev = f->prev,*next = f->next;
	if(prev != NULL && prev->is_used == false)
	{
		prev->size = prev->size + f->size;
		prev->next = f->next;
		f = prev; 
	}
	if(next != NULL && next->is_used == false)
	{
		f->size = f->size + next->size;
		f->next = next->next;
	}
}
int main()
{
	mm_init(0,1024);
	cout << "before malloc the memory is " << endl;
	print_block();
	cout << endl;
	cout << "最差匹配算法，先分配100，在分配200，释放100，再分配50" << endl;
	block *a1 = malloc(100);
	block *a2 = malloc(200);
	print_block();
	free(a1);
	print_block();
	malloc(50);
	print_block();
	return 0;
}
```
> 运行结果如下，可以发现当有100和724两个空闲快同时存在的时候，分配50的时候选择了后者

```
before malloc the memory is
start location = 0 size = 1024 is_used = 0

最差匹配算法，先分配100，在分配200，释放100，再分配50
start location = 0 size = 100 is_used = 1
start location = 100 size = 200 is_used = 1
start location = 300 size = 724 is_used = 0
start location = 0 size = 100 is_used = 0
start location = 100 size = 200 is_used = 1
start location = 300 size = 724 is_used = 0
start location = 0 size = 100 is_used = 0
start location = 100 size = 200 is_used = 1
start location = 300 size = 50 is_used = 1
start location = 350 size = 674 is_used = 0

--------------------------------
Process exited with return value 0
Press any key to continue . . .
```

---

## 扩展思考题

阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次
MMU的工作机理？

- [x]  

>  http://en.wikipedia.org/wiki/Memory_management_unit

L1和L2高速缓存有什么区别？

- [x]  

>  http://superuser.com/questions/196143/where-exactly-l1-l2-and-l3-caches-located-in-computer
>  Where exactly L1, L2 and L3 Caches located in computer?

>  http://en.wikipedia.org/wiki/CPU_cache
>  CPU cache

### 5.2 地址空间和地址生成
编译、链接和加载的过程了解？

- [x]  

>  

动态链接如何使用？

- [x]  

>  


### 5.3 连续内存分配
什么是内碎片、外碎片？

- [x]  

>  

为什么最先匹配会越用越慢？

- [x]  

>  

为什么最差匹配会的外碎片少？

- [x]  

>  

在几种算法中分区释放后的合并处理如何做？

- [x]  

>  

### 5.4 碎片整理
一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

- [x]  

>  

### 5.5 伙伴系统
伙伴系统的空闲块如何组织？

- [x]  

>  

伙伴系统的内存分配流程？

- [x]  

>  

伙伴系统的内存回收流程？

- [x]  

>  

struct list_entry是如何把数据元素组织成链表的？

- [x]  

>  

