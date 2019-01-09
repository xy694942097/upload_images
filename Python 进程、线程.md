### 1. 进程
> 程序本身只是指令、数据及其组织形式的描述，进程才是程序（那些指令和数据）的真正运行实例。

进程是执行中的程序。

进程在运行时，状态（state）会改变。这是为了进程间的切换。
一个进程具有：就绪、运行、中断、僵死、结束等状态（不同操作系统不一样）

既然操作系统不可能一直执行某个进程，所以进程当然有各种不同的状态。
![在这里插入图片描述](https://img-blog.csdn.net/20160906192211991)

特性：
* 每个程序，本身首先是一个进程
* 运行中每个进程都拥有自己的地址空间、内存、数据栈及其它资源。
* 操作系统本身自动管理着所有的进程(不需要用户代码干涉)，并为这些进程合理分配可以执行时间。
* 进程可以通过派生新的进程来执行其它任务，不过每个进程还是都拥有自己的内存和数据栈等。
* 进程间可以通讯(发消息和数据)，采用 进程间通信(IPC) 方式。
* 多个进程可以在不同的 CPU 上运行，互不干扰
* 同一个CPU上，可以运行多个进程，由操作系统来自动分配时间片
* 由于进程间资源不能共享，需要进程间通信，来发送数据，接受消息等

多进程，也称为“并行”。？
多进程可以充分利用多核CPU。

进程锁。如果多个进程同时请求同一个资源会造成混乱。

### 2. 线程
线程是操作系统调度的最小单位。
线程包含在进程中，一个进程可以并发多个线程，执行不同任务。

使用

用户编写包含线程的程序(每个程序本身都是一个进程)
操作系统“程序切换”进入当前进程
当前进程包含了线程，则启动线程
多个线程，则按照顺序执行，除非抢占
特性？？？？？？？？？

进程和线程的区别

一个进程中的各个线程与主进程共享相同的资源，与进程间互相独立相比，线程之间信息共享和通信更加容易(都在进程中，并且共享内存等)。

线程一般以并发执行，正是由于这种并发和数据共享机制，使多任务间的协作成为可能。

进程一般以并行执行，这种并行能使得程序能同时在多个CPU上运行；

区别于多个线程只能在进程申请到的的“时间片”内运行(一个CPU内的进程，启动了多个线程，线程调度共享这个进程的可执行时间片)，进程可以真正实现程序的“同时”运行(多个CPU同时运行)。

进程和线程的常用应用场景

一般来说,在Python中编写并发程序的经验：

计算密集型任务使用多进程
IO密集型(如：网络通讯)任务使用多线程，较少使用多进程.
这是由于 IO操作需要独占资源，比如：

网络通讯(微观上每次只有一个人说话，宏观上看起来像同时聊天)每次只能有一个人说话
文件读写同时只能有一个程序操作(如果两个程序同时给同一个文件写入 'a', 'b'，那么到底写入文件的哪个呢？)
都需要控制资源每次只能有一个程序在使用，在多线程中，由主进程申请IO资源，多线程逐个执行，哪怕抢占了，也是逐个运行，感觉上“多线程”并发执行了。

如果多进程，除非一个进程结束，否则另外一个完全不能用，显然多进程就“浪费”资源了。



线程，必须在一个存在的进程中启动运行
线程使用进程获得的系统资源，不会像进程那样需要申请CPU等资源
线程无法给予公平执行时间，它可以被其他线程抢占，而进程按照操作系统的设定分配执行时间
每个进程中，都可以启动很多个线程



### 3. 协程

协程，也是”程序切换“的一种。

这里提一个特殊的“线程”，也就是协程的概念。

定义

简单说，协程也是线程，只是协程的调度并不是由操作系统调度，而是自己”协同调度“。也就是”协程是不通过操作系统调度的线程“。



并发：看上去一起执行，任务多于核心
并行：真正一起执行，任务小于核心数

总结
1. 进程，线程是操作系统级的，协程是语言级的。 
2. 每个进程至少包含一个线程，因为线程才是真正的运行单位。 
3. 线程进程都是同步机制，而协程则是异步。 
4. IO密集型一般使用多线程或者多进程，CPU密集型一般使用多进程，强调非阻塞异步并发的一般都是使用协程，当然有时候也是需要多进程线程池结合的，或者是其他组合方式。


### 代码-进程
首先对于单任务，如果你连续写2个死循环，那么第2个循环永远不会执行，因为单任务必须是顺序执行。
```python
while 1:
	print('loop 1')
	time.sleep(1)
while 1:						# 循环2永远不会执行
	print('loop 2')
	time.sleep(2)
```

如果是多任务，那么2个死循环都可以执行。
```python
import multiprocessing, time, os
def func(i):
	while 1:
		print("执行子进程: pid=%s, ppid=%s"%(os.getpid(),os.getppid()))
		time.sleep(1)
if __name__ == '__main__':
	print("主进程开始: pid=%s, ppid=%s"%(os.getpid(),os.getppid()))
	p = multiprocessing.Process(target=func, args=('son',))
	p.start()

	while 1:
		print("执行主进程,pid=%s, ppid=%s"%(os.getpid(),os.getppid()))
		time.sleep(1)

	print("父进程结束.")		# 这一句实在主进程的loop之后，由于loop不会结束，所以不会执行到。
```
输出如下：
```
主进程开始: pid=4824, ppid=12508
执行主进程,pid=4824, ppid=12508
执行子进程: pid=5508, ppid=4824
执行主进程,pid=4824, ppid=12508
执行子进程: pid=5508, ppid=4824
...
```
此时查看任务管理器：
![在这里插入图片描述](https://img-blog.csdn.net/20180923102042467?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXdlaXhpYW8z/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
可以看到，python主程序首先是一个进程，一般称之为主进程，这里pid是4824，它创建了另一个进程5508，是主进程的子进程，而主进程也有父进程是windows命令处理程序，也就是黑窗口。
可以看到的是这2个infinite loop我们分别放在主进程和子进程中，它都能被执行。

**父子进程先后顺序：**
我们在父进程放一个for loop，在2个子进程中也放for loop看看：
```python
import multiprocessing, time, os
def func(name):
	for i in range(3):
		print("执行子进程%s: pid=%s, ppid=%s"%(name,os.getpid(),os.getppid()))
		time.sleep(1)
	print('子进程%s结束.'%name)
if __name__ == '__main__':
	print("主进程开始: pid=%s"%os.getpid())
	p1 = multiprocessing.Process(target=func, args=('1',))
	p1.start()
	
	p2 = multiprocessing.Process(target=func, args=('2',))
	p2.start()

	for i in range(3):
		print("执行主进程,pid=%s"%os.getpid())
		time.sleep(1)

	print("主进程结束.")
```
输出如下：
```
主进程开始: pid=5032
执行主进程,pid=5032
执行子进程1: pid=1344, ppid=5032
执行子进程2: pid=2596, ppid=5032
执行主进程,pid=5032
执行子进程1: pid=1344, ppid=5032
执行子进程2: pid=2596, ppid=5032
执行主进程,pid=5032
执行子进程1: pid=1344, ppid=5032
执行子进程2: pid=2596, ppid=5032
主进程结束.
子进程1结束.
子进程2结束.
```
可以看到，我主进程和子进程都是3个循环时，主进程比子进程先结束，而且主进程结束后子进程还在继续执行。

有时候我们希望主进程在所有子进程结束后才结束，因为一般任务都是给子进程做的，主进程看着就行了。使用`join`方法让主进程等待子进程。
`p.join(timeout=None) wait until child process terminates.` 

实际上让主进程等待子进程，就是先执行子进程，子进程结束后再执行主进程。

如上例中在`p1.start()`后加`p1.join()`，那么会先执行子进程1，子进程1结束后，子进程2和主进程还是交替/同时执行，因为子进程2没有使用`join`方法.

如果只给进程2加`join`方法呢，则是进程1和2交替执行完，再执行主进程。主进程要等待子进程2完成，但是子进程1和子进程2是没有先后顺序的。

如果同时给进程1和2加`join`方法，则时先执行完进程1，再进程2，再主进程。不过如果给2个子进程都加`join`方法，那和只有1个进程就没什么区别了吧。 

**全局变量在多个进程中不能共享**
在子进程中修改全局变量对父进程中的全局变量没有影响。兄弟进程之间也不共享。
在创建子进程时对全局变量进行了备份，父子进程中的变量时2个不同的变量。

**启动大量进程：进程池**
```python
from multiprocessing import Pool
import os, time, random
def task(name):
    print('Run task %s (%s)...' % (name, os.getpid()))
    start = time.time()
    time.sleep(random.random() * 3)
    end = time.time()
    print('Task %s runs %0.2f seconds.' % (name, (end - start)))

if __name__=='__main__':
    print('Parent process %s.' % os.getpid())
    p = Pool(4)		# 创建进程池，同时执行4个进程，默认时cpu核心数
    for i in range(5):
        p.apply_async(task, args=(i,))	# 给进程池添加进程。这里放了5个进程
    print('Waiting for all subprocesses done...')
    p.close()
    p.join()		# 等待所有子进程完成，一定要先关闭进程池。
    print('All subprocesses done.')
```
这里task 0/1/2/3会立刻执行，因为核心数是4，然后随机执行完1个才执行task 5。

拷贝大量文件时可以用多进程加快效率。
少量时反而降低，因为线程的切换等等是要耗时的。

**进程间通信**
```python
from multiprocessing import Process, Queue
import os, time, random

# 写数据进程执行的代码:
def write(q):
    print('Process to write: %s' % os.getpid())
    for value in ['A', 'B', 'C']:
        print('Put %s to queue...' % value)
        q.put(value)
        time.sleep(random.random())

# 读数据进程执行的代码:
def read(q):
    print('Process to read: %s' % os.getpid())
    while True:
        value = q.get(True)
        print('Get %s from queue.' % value)

if __name__=='__main__':
    # 父进程创建Queue，并传给各个子进程：
    q = Queue()
    pw = Process(target=write, args=(q,))
    pr = Process(target=read, args=(q,))
    # 启动子进程pw，写入:
    pw.start()
    # 启动子进程pr，读取:
    pr.start()
    # 等待pw结束:
    pw.join()
    # pr进程里是死循环，无法等待其结束，只能强行终止:
    pr.terminate()
```
### 代码-线程
`threading`模块
```python
import time, threading

# 新线程执行的代码:
def loop():
    print('thread %s is running...' % threading.current_thread().name)	# threading.current_thread永远返回当前线程，任何线程默认有个线程-主线程MainThread
    n = 0
    while n < 5:
        n = n + 1
        print('thread %s >>> %s' % (threading.current_thread().name, n))
        time.sleep(1)
    print('thread %s ended.' % threading.current_thread().name)

print('thread %s is running...' % threading.current_thread().name)
t = threading.Thread(target=loop)
t.start()
t.join()			# 让主线程等待子线程结束，或者说是让该子线程执行完再执行之后的代码
print('thread %s ended.' % threading.current_thread().name)
```
**Lock**
多进程中，同一个变量，各自有一份拷贝存在于每个进程中，互不影响，而多线程中，所有变量都由所有线程共享，所以，任何一个变量都可以被任何一个线程修改，因此，线程之间共享数据最大的危险在于多个线程同时改一个变量，把内容给改乱了。

[例子](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/00143192823818768cd506abbc94eb5916192364506fa5d000)
其原因在于赋值语句`x+=5`在高级语言中也分2步：`tmp=x+5;x=tmp`，用到一个临时变量，而临时变量在2个线程中是各自的。线程的执行不是顺序执行的，有可能中断，所以顺序乱了可能会改乱一个对象。

给修改函数加把锁，使其不能被多个线程同时执行，只能被1个线程执行，就可以保证全局变量的值不会被改乱。
```python
balance = 0
lock = threading.Lock()		# 初始化锁
def run_thread(n):
    for i in range(100000):
        lock.acquire()		# 上锁
        try:
            change_it(n)	# 修改
        finally:
            lock.release()	# 释放锁
```
当多个线程执行`lock.acquire()`时，只有1个线程能获取锁，没有锁的线程就只能等着，保证关键函数只能被1个线程执行。

获得锁的线程用完后一定要释放锁，用try...finally来确保锁一定会被释放。

锁的好处就是确保了某段关键代码只能由一个线程从头到尾完整地执行，坏处当然也很多，首先是阻止了多线程并发执行，包含锁的某段代码实际上只能以单线程模式执行，效率就大大地下降了。其次，由于可以存在多个锁，不同的线程持有不同的锁，并试图获取对方持有的锁时，可能会造成死锁，导致多个线程全部挂起，既不能执行，也无法结束，只能靠操作系统强制终止。

### python线程的局限
Python的线程虽然是真正的线程，但解释器执行代码时，有一个GIL锁：Global Interpreter Lock，任何Python线程执行前，必须先获得GIL锁，然后，每执行100条字节码，解释器就自动释放GIL锁，让别的线程有机会执行。这个GIL全局锁实际上把所有线程的执行代码都给上了锁，所以，多线程在Python中只能交替执行，即使100个线程跑在100核CPU上，也只能用到1个核。


Python虽然不能利用多线程实现多核任务，但可以通过多进程实现多核任务。多个Python进程有各自独立的GIL锁，互不影响。

==python的threading多线程只是个错觉==

### 计算密集型 vs. IO密集型
是否采用多任务的第二个考虑是任务的类型。我们可以把任务分为计算密集型和IO密集型。

计算密集型任务的特点是要进行大量的计算，消耗CPU资源，比如计算圆周率、对视频进行高清解码等等，全靠CPU的运算能力。这种计算密集型任务虽然也可以用多任务完成，但是任务越多，花在任务切换的时间就越多，CPU执行任务的效率就越低，所以，要最高效地利用CPU，计算密集型任务同时进行的数量应当等于CPU的核心数。

计算密集型任务由于主要消耗CPU资源，因此，代码运行效率至关重要。Python这样的脚本语言运行效率很低，完全不适合计算密集型任务。对于计算密集型任务，最好用C语言编写。

第二种任务的类型是IO密集型，涉及到网络、磁盘IO的任务都是IO密集型任务，这类任务的特点是CPU消耗很少，任务的大部分时间都在等待IO操作完成（因为IO的速度远远低于CPU和内存的速度）。对于IO密集型任务，任务越多，CPU效率越高，但也有一个限度。常见的大部分任务都是IO密集型任务，比如Web应用。

IO密集型任务执行期间，99%的时间都花在IO上，花在CPU上的时间很少，因此，用运行速度极快的C语言替换用Python这样运行速度极低的脚本语言，完全无法提升运行效率。对于IO密集型任务，最合适的语言就是开发效率最高（代码量最少）的语言，脚本语言是首选，C语言最差。

### 分布式进程
[添加链接描述](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431929340191970154d52b9d484b88a7b343708fcc60000)


。
。

。

。
。

。
。
。
。
。
。
。

。
。
。
。
。

。。

。
。
。
。
多线程
threading模块可以创建多个线程，不过由于GIL锁的存在，Python在多线程里面其实是快速切换
```python
class threading.Thread(
	group=None, 	# 为None就行。
	target=None, 	# 线程要执行的函数，被run()调用
	name=None, 		# 线程名字
	args=(), 		# 给target的元组参数
	kwargs={}, 		# 给target的字典参数
	*, daemon=None)	# daemon意为守护进程,默认从当前线程继承，
```

```python
import time
def f(i):
    print(i)
    time.sleep(1)
    print(i)

for i in range(15):
    p = threading.Thread(target=f,args=(i,))
    p.start()
```
#默认情况下程序会等线程全部执行完毕才停止的，不过可以设置更改为后台线程，使主线程不等待子线程，主线程结束则全部结束













sdfsdf







```python
import multiprocessing
import time
def run(i):		# 子进程执行的方法
	time.sleep(1)
	print(i)

for i in range(10):
	p = multiprocessing.Process(target=run, args=(i))	# 创建子进程，指定子进程执行程序及其参数
	p.start()	# 运行子进程
```
