# 同步互斥(lec 18) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 基本理解
 - 什么是信号量？它与软件同步方法的区别在什么地方？
 - 什么是自旋锁？它为什么无法按先来先服务方式使用资源？
 - 下面是一种P操作的实现伪码。它能按FIFO顺序进行信号量申请吗？
```
 while (s.count == 0) {  //没有可用资源时，进入挂起状态；
        调用进程进入等待队列s.queue;
        阻塞调用进程;
}
s.count--;              //有可用资源，占用该资源； 
```

> 参考回答： 它的问题是，不能按FIFO进行信号量申请。
> 它的一种出错的情况
```
一个线程A调用P原语时，由于线程B正在使用该信号量而进入阻塞状态；注意，这时value的值为0。
线程B放弃信号量的使用，线程A被唤醒而进入就绪状态，但没有立即进入运行状态；注意，这里value为1。
在线程A处于就绪状态时，处理机正在执行线程C的代码；线程C这时也正好调用P原语访问同一个信号量，并得到使用权。注意，这时value又变回0。
线程A进入运行状态后，重新检查value的值，条件不成立，又一次进入阻塞状态。
至此，线程C比线程A后调用P原语，但线程C比线程A先得到信号量。
```

### 信号量使用

 - 什么是条件同步？如何使用信号量来实现条件同步？
 - 什么是生产者-消费者问题？
 - 为什么在生产者-消费者问题中先申请互斥信息量会导致死锁？

### 管程

 - 管程的组成包括哪几部分？入口队列和条件变量等待队列的作用是什么？
 - 为什么用管程实现的生产者-消费者问题中，可以在进入管程后才判断缓冲区的状态？
 - 请描述管程条件变量的两种释放处理方式的区别是什么？条件判断中while和if是如何影响释放处理中的顺序的？

### 哲学家就餐问题

 - 哲学家就餐问题的方案2和方案3的性能有什么区别？可以进一步提高效率吗？

### 读者-写者问题

 - 在读者-写者问题的读者优先和写者优先在行为上有什么不同？
 - 在读者-写者问题的读者优先实现中优先于读者到达的写者在什么地方等待？
 
## 小组思考题

1. （spoc） 每人用python threading机制用信号量和条件变量两种手段分别实现[47个同步互斥问题](07-2-spoc-pv-problems.md)中的一题。向勇老师的班级从前往后，陈渝老师的班级从后往前。请先理解[]python threading 机制的介绍和实例](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/semaphore_condition)


>针对第五个问题，

大致输出如下：
```
I am seller. 
Selling paper and matches. 
I am smoker 2. 
I am smoker 3. I am smoker 1. 

getting paper and matches. 
smoker 2 is smoking. 
Selling tobacco and matches. 
getting tabacco and matches. 
smoker 3 is smoking. 
Selling paper and tobacco. 
getting paper and tabacco. 
smoker 1 is smoking. 
Selling tobacco and matches. 
getting tabacco and matches. 
smoker 3 is smoking. 
Selling paper and matches. 
getting paper and matches. 
smoker 2 is smoking. 
Selling tobacco and matches. 
getting tabacco and matches. 
smoker 3 is smoking. 

```

>给出如下代码：
>代码如下（信号量）：
>

```
import threading
import time
import random

class Seller(threading.Thread):
	def __init__(self, threadName, s, sd1, sd2, sd3):
		threading.Thread.__init__(self, name=threadName)
		self.sleepTime = 1
		self.s = s
		self.sd1 = sd1
		self.sd2 = sd2
		self.sd3 = sd3

	def run(self):
		while True:
			self.s.acquire()
			r = random.randint(1, 3)
			if r == 1:
				print 'Selling paper and matches. '
				self.sd1.release()
			elif r == 2:
				print 'Selling tobacco and matches. '
				self.sd2.release()
			elif r == 3:
				print 'Selling paper and tobacco. '
				self.sd3.release()
			self.s.release()
			time.sleep(self.sleepTime)

class Smoker(threading.Thread):
	def __init__(self, threadName, s, sabc):
		threading.Thread.__init__(self, name=threadName)
		self.sleepTime = 1
		self.s = s
		self.sabc = sabc

	def run(self):
		while True:
			self.sabc.acquire()
			if self.name == 'Somker_A':
				print 'getting paper and matches. '
			elif self.name == 'Somker_B':
				print 'getting tabacco and matches. '
			else:
				print 'getting paper and tabacco. '
			self.s.release()
			print '%s is smoking. ' %(self.name)
			time.sleep(self.sleepTime)

s = threading.Semaphore(1)
sd1 = threading.Semaphore(0)
sd2 = threading.Semaphore(0)
sd3 = threading.Semaphore(0)

seller = Seller("Seller", s, sd1, sd2, sd3)
smoker1 = Smoker("Somker_A", s, sd1)
smoker2 = Smoker("Somker_B", s, sd2)
smoker3 = Smoker("Somker_C", s, sd3)

seller.start()
smoker1.start()
smoker3.start()
smoker2.start()
```

>代码如下（条件变量）

```
import threading 
import time
import random

condition = threading.Condition()
flag = 0

class Seller(threading.Thread):
	def __init__(self, threadName):
		threading.Thread.__init__(self, name=threadName)

	def run(self):
		global condition, flag
		print 'I am seller. '
		while True:
			if condition.acquire():
				if flag == 0:
					flag = random.randint(1, 3)
					if flag == 1:
						print 'Selling paper and matches. '
					elif flag == 2:
						print 'Selling tobacco and matches. '
					elif flag == 3:
						print 'Selling paper and tobacco. '
					condition.notifyAll()
				else:
					condition.wait()
				condition.release()

class Smoker(threading.Thread):
	def __init__(self, threadName, index):
		threading.Thread.__init__(self, name=threadName)
		self.index = index

	def run(self):
		global condition, flag
		print 'I am smoker %d. ' %(self.index)
		while True:
			if condition.acquire():
				if flag == self.index:
					if self.index == 1:
						print 'getting paper and matches. '
					elif self.index == 2:
						print 'getting tabacco and matches. '
					elif self.index == 3:
						print 'getting paper and tabacco. '
					print '%s is smoking. ' %(self.name)
					time.sleep(1)
					flag = 0
					condition.notifyAll()
			else:
				condition.wait()
			condition.release()


seller = Seller('Seller')
smoker1 = Smoker('somker1', 1)
smoker2 = Smoker('somker2', 2)
smoker3 = Smoker('somker3', 3)

seller.start()
smoker1.start()
smoker2.start()
smoker3.start()
```




2. (spoc)设计某个方法，能够动态检查出对于两个或多个进程的同步互斥问题执行中，没有互斥问题，能够同步等，以说明实现的正确性。

>在临界区内修改一个信号量，在进入时自增，退出时自减。如果在进入时发现变量已满，那么说明进程的同步问题出错了。

