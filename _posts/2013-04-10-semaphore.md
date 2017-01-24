---
layout: post
title : 信号量简单应用
category : program
---
{% include JB/setup %}

什么是信号量?简单的说就是,很多干女儿要找干爹,但是干爹只有一个,所以干爹一次只能满足一个干儿女的要求,那么干儿女的先后顺序怎么办呢?很好解决,谁收到干爹的短信,谁先来找干爹.

哈哈,来看下百度百科的解释:

	信号量(Semaphore)，有时被称为信号灯，是在多线程环境下使用的一种设施，是可以用来保证两个或多个关键代码段不被并发调用。在进入一个关键代码段之前，线程必须获取一个信号量；一旦该关键代码段完成了，那么该线程必须释放信号量。其它想进入该关键代码段的线程必须等待直到第一个线程释放信号量。为了完成这个过程，需要创建一个信号量VI，然后将Acquire Semaphore VI以及Release Semaphore VI分别放置在每个关键代码段的首末端。确认这些信号量VI引用的是初始创建的信号量。

下面给出几个简单的信号量问题,已经问题的解决方法(如果有错误,欢迎指正)

### 哲学家就餐问题

哲学家就餐问题可以这样表述，假设有五位哲学家围坐在一张圆形餐桌旁，做以下两件事情之一：吃饭，或者思考。吃东西的时候，他们就停止思考，思考的时候也停止吃东西。餐桌中间有一大碗意大利面，每两个哲学家之间有一只餐叉。因为用一只餐叉很难吃到意大利面，所以假设哲学家必须用两只餐叉吃东西。他们只能使用自己左右手边的那两只餐叉。哲学家就餐问题有时也用米饭和筷子而不是意大利面和餐叉来描述，因为很明显，吃米饭必须用两根筷子。

哲学家从来不交谈，这就很危险，可能产生死锁，每个哲学家都拿着左手的餐叉，永远都在等右边的餐叉（或者相反）。即使没有死锁，也有可能发生资源耗尽。例如，假设规定当哲学家等待另一只餐叉超过五分钟后就放下自己手里的那一只餐叉，并且再等五分钟后进行下一次尝试。这个策略消除了死锁（系统总会进入到下一个状态），但仍然有可能发生“活锁”。如果五位哲学家在完全相同的时刻进入餐厅，并同时拿起左边的餐叉，那么这些哲学家就会等待五分钟，同时放下手中的餐叉，再等五分钟，又同时拿起这些餐叉。

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<semaphore.h>
#include<pthread.h>
#define NUM 5

sem_t chopstick[NUM];

pthread_t thinker[NUM];

void * eat(void *arg){
	int num = *(int *)arg;
	while(1){
		if(num%2 == 0){
			sem_wait(num == (NUM-1) ? &chopstick[0]:&chopstick[num+1]);
			sem_wait(&chopstick[num]);
			printf("thinker %d is eating\n",num);
			sem_post(num == (NUM-1) ? &chopstick[0]:&chopstick[num+1]);
			sem_post(&chopstick[num]);			
		}else{
			sem_wait(&chopstick[num]);
			sem_wait(&chopstick[num+1]);		
			printf("thinker %d is eating\n",num);
			sem_post(&chopstick[num]);
			sem_post(&chopstick[num+1]);
		}
	}
	return((void *)0);
}

int main(void){
	int err[NUM];
	int num[5] = {0,1,2,3,4};
	int i;
	for(i=0;i<NUM;i++){
		sem_init(&chopstick[i],0,1);
	}
	for(i=0;i<NUM;i++){
		err[i] = pthread_create(&thinker[i],NULL,eat,&num[i]);
		if(err[i] != 0){
			printf("can't create thread: %s\n",strerror(err[i]));
		}
	}
	while(!getchar());
	return 0;
}
```

### 简单互斥问题 

四个进程A、B、C、D都要读一个共享文件F，系统允许多个进程同时读文件F。但限制是进程A和进程C不能同时读文件F，进程B和进程D也不能同时读文件F。为了使这四个进程并发执行时能按系统要求使用文件，现用P、V操作进行管理 

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<semaphore.h>
#include<pthread.h>

//A和C的信号量
sem_t ac;
//B和D的信号量
sem_t bd;
//ABCD4个进程
pthread_t abcd[4];

void * read(void *arg){
	char who = *(char *)arg;
	while(1){
		if(who=='A'||who=='C'){
			sem_wait(&ac);
			printf("%c is reading\n",who);
			printf("%c finished reading\n",who);
			sem_post(&ac);		
		}else{
			sem_wait(&bd);
			printf("%c is reading\n",who);
			printf("%c finished reading\n",who);
			sem_post(&bd);	
		}
	}
	return((void *)0);
}

int main(void){
	int err[4];
	char who[4] = {'A','B','C','D'};
	int i;

	sem_init(&ac,0,1);
	sem_init(&bd,0,1);

	for(i=0;i<4;i++){
		err[i] = pthread_create(&abcd[i],NULL,read,&who[i]);
		if(err[i] != 0){
			printf("can't create thread: %s\n",strerror(err[i]));
		}
	}
	while(!getchar());
	return 0;
}
```

### 阅览室问题

有一阅览室，读者进入时必须先在一张登记表上进行登记，该表为每一座位列一表目，包括座号和读者姓名。读者离开时要消掉登记信号，阅览室中共有100个座位。用PV操作控制这个过程

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<semaphore.h>
#include<pthread.h>
//阅览室读者容纳信号量
sem_t countMutex;
//阅览室读者座位信息登记和注销信号量
sem_t seatMutex;
//阅览室座位编号,值0表示座位空闲,值1表示座位已用
int seatId[100];
//读者线程
pthread_t reader[200];

//获取座位号
int getSid(){
	int i;
	for(i=0;i<100;i++){
		if(seatId[i] == 0){
			seatId[i] = 1;
			return i;
		}	
	}
}

//释放座位号
void releaseSid(int sid){
	seatId[sid] = 0;
}

void * read(void *arg){
	int num = *(int *)arg;
	int count = 0;
	int sid = -1;
	//while(count<5){

		sem_wait(&countMutex);

		sem_wait(&seatMutex);
		sid = getSid();
		sem_post(&seatMutex);

		printf("Reader%d get seat %d\n",num,sid);
		usleep(100);
		printf("Reader%d leave seat %d\n",num,sid);

		sem_wait(&seatMutex);
		releaseSid(sid);
		sem_post(&seatMutex);

		count++;
		sem_post(&countMutex);		
	//}
	return((void *)0);
}

int main(void){
	int err[200];
	int num[200];
	int i;
	
	sem_init(&countMutex,0,100);
	sem_init(&seatMutex,0,1);

	for(i=0;i<100;i++){
		seatId[i] = 0;	
	}

	for(i=0;i<200;i++){
		num[i] = i;
		err[i] = pthread_create(&reader[i],NULL,read,&num[i]);
		if(err[i] != 0){
			printf("can't create thread: %s\n",strerror(err[i]));
		}
	}

	while(!getchar());
	return 0;
}
```

### 理发师问题

理发店理有一位理发师、一把理发椅和n把供等候理发的顾客坐的椅子.如果没有顾客，理发师便在理发椅上睡觉。一个顾客到来时，它必须叫醒理发师.如果理发师正在理发时又有顾客来到，则如果有空椅子可坐，就坐下来等待，否则就离开。

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<semaphore.h>
#include<pthread.h>
//设定5个等候座位
#define WAIT_SEAT_NUM 5
//理发师信号量
sem_t barberMutex;
//顾客数信号量
sem_t customerCountMutex;
//理发师状态信号量
sem_t barberStatus;
//顾客计数器
int customerCount;
//顾客线程
pthread_t customer[20];
//理发师线程
pthread_t barber;

void * Customer(void *arg){
	int num = *(int *)arg;

		sem_wait(&customerCountMutex);
			if(customerCount == WAIT_SEAT_NUM + 1){
				/*
				*如果顾客计数器等于6,则人满离开
				*/
				printf("because there is no wait seat,customer%d leaves\n",num);
				sem_post(&customerCountMutex);
				return((void *)0);
			}
		customerCount++;
		/*
		*第一个顾客则更改理发师状态
		*/
		if(customerCount == 1){
			sem_wait(&barberStatus);
			printf("customer%d want a haircut,so he is waking barber up\n",num);
		}else{	
			/*
			*如果加上顾客本身计数器大于1,则就坐等候
			*/
			printf("customer%d is waitting on a wait seat\n",num);
		}		
		sem_post(&customerCountMutex);
		/*
		*等待空闲理发师
		*有空闲理发师了,则开始理发
		*理完发后,顾客计算器减1
		*如果此时没有顾客,则改变理发师状态
		*/
		sem_wait(&barberMutex);
			printf("customer%d is cutting\n",num);
			usleep(2);
			printf("customer%d is finished\n",num);
			sem_wait(&customerCountMutex);
			customerCount--;
			if(customerCount == 0){
				sem_post(&barberStatus);
			}
			sem_post(&customerCountMutex);
		sem_post(&barberMutex);

	return((void *)0);
}

void * Barber(void *arg){
	/*
	*没有顾客则睡觉
	*/
	while(1){
		sem_wait(&barberStatus);
			printf("because there is no customer,barber is sleeping\n");
			sleep(2);
		sem_post(&barberStatus);
	}
	return((void *)0);
}


int main(void){
	int err[20];
	int num[20];
	customerCount = 0;
	int i;

	sem_init(&barberMutex,0,1);
	sem_init(&customerCountMutex,0,1);
	sem_init(&barberStatus,0,1);

	for(i=0;i<20;i++){
		num[i] = i;
		err[i] = pthread_create(&customer[i],NULL,Customer,&num[i]);
		if(err[i] != 0){
			printf("can't create thread: %s\n",strerror(err[i]));
		}
	}
	
	i = pthread_create(&barber,NULL,Barber,NULL);
	if(i != 0){
		printf("can't create thread: %s\n",strerror(i));
	}
	
	while(!getchar());
	return 0;
}
```

### 多理发师问题

Three chairs.Three barbers.Waiting area having capacity for four seats and additional space for customers to stand and wait.Fire code limits the total number of customers (seating +standing).Customer cannot enter if pack to capacity.When a barber is free, customer waiting longest on the sofa can be served.If there are any standing customer, he can now occupy the seat on the sofa.Once a barber finished haircut, any barber can take the payment.As there is only one cash register, so payment from one customer is accepted at a time.Barber(s) time is divided into haircut, accepting payment and waiting for customers (sleeping).

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<semaphore.h>
#include<pthread.h>
//4个等候座位
#define WAIT_SEAT_NUM 4
//6个站立等候空间
#define STAND_NUM 6
//顾客容纳总数
#define TOTAL_NUM 13

//理发师信号量
sem_t barberMutex;
//座位信号量
sem_t seat;
//顾客数信号量
sem_t customerCountMutex;
//收银机信号量
sem_t cashRegister;
//顾客计数器
int customerCount;
//顾客线程
pthread_t customer[40];
//理发师线程
pthread_t barber[3];

void * Customer(void *arg){
	//顾客编号
	int num = *(int *)arg;

		sem_wait(&customerCountMutex);
			if(customerCount == TOTAL_NUM){
				/*
				*如果顾客计数器等于13,则人满离开
				*/
				printf("because there is no stand space,customer%d leaves\n",num);
				sem_post(&customerCountMutex);
				return((void *)0);
			}else if(customerCount >= WAIT_SEAT_NUM + 3){
				/*
				*如果顾客计数器在7~12,则站立等候,顾客计数器加1,并等待座位
				*有座位了就入座,并等待空闲理发师
				*有空闲理发师了,则开始理发
				*理完发后等候收银机
				*收银机空闲则付款,顾客计算器减1
				*/
				printf("because there is no wait seat,customer%d stands to wait\n",num);
				customerCount++;
				sem_post(&customerCountMutex);
				sem_wait(&seat);
					printf("because someone leaves wait seat to hava a haircut,customer%d get a wait seat\n",num);	
				sem_wait(&barberMutex);
					printf("customer%d leaves wait seat to hava a haircut\n",num);
					sem_post(&seat);										
					usleep(10);
					printf("customer%d is finished\n",num);
					sem_wait(&cashRegister);
						printf("customer%d payed money\n",num);
					sem_post(&cashRegister);	
					sem_wait(&customerCountMutex);
						customerCount--;
					sem_post(&customerCountMutex);	
				sem_post(&barberMutex);	
				return((void *)0);
			}else if(customerCount >= 3){
				/*
				*如果顾客计数器在3~6,则坐着等候,顾客计数器加1,并等待空闲理发师
				*有空闲理发师了,则开始理发
				*理完发后等候收银机
				*收银机空闲则付款,顾客计算器减1
				*/
				printf("customer%d is waitting on a wait seat\n",num);
				customerCount++;
				sem_post(&customerCountMutex);
				sem_wait(&seat);
				sem_wait(&barberMutex);
					printf("customer%d leaves wait seat to hava a haircut\n",num);
					sem_post(&seat);
					usleep(10);
					printf("customer%d is finished\n",num);
					sem_wait(&cashRegister);
						printf("customer%d payed money\n",num);
					sem_post(&cashRegister);	
					sem_wait(&customerCountMutex);
						customerCount--;
					sem_post(&customerCountMutex);	
				sem_post(&barberMutex);
				return((void *)0);
			}else{
				/*
				*如果顾客计数器小于3,则开始理发
				*理完发后等候收银机
				*收银机空闲则付款,顾客计算器减1
				*/
				customerCount++;
				sem_post(&customerCountMutex);
				sem_wait(&barberMutex);
					printf("customer%d is cutting\n",num);
					usleep(10);
					printf("customer%d is finished\n",num);
					sem_wait(&cashRegister);
						printf("customer%d payed money\n",num);
					sem_post(&cashRegister);		
					sem_wait(&customerCountMutex);
						customerCount--;
					sem_post(&customerCountMutex);	
				sem_post(&barberMutex);						
			}
	return((void *)0);
}

void * Barber(void *arg){
	//理发师编号
	int num = *(int *)arg;
	/*
	*没有顾客则睡觉
	*/
	while(1){
		sem_wait(&barberMutex);
			printf("because there is no enough customer ,barber%d is sleeping\n",num);
			sleep(2);
		sem_post(&barberMutex);
	}
	return((void *)0);
}
int main(void){
	int err[40];
	int num[40];
	customerCount = 0;
	int i;
	//有3个理发师
	sem_init(&barberMutex,0,3);
	//有4个座位
	sem_init(&seat,0,4);
	sem_init(&customerCountMutex,0,1);
	sem_init(&cashRegister,0,1);

	for(i=0;i<40;i++){
		num[i] = i;
		err[i] = pthread_create(&customer[i],NULL,Customer,&num[i]);
		if(err[i] != 0){
			printf("can't create thread: %s\n",strerror(err[i]));
		}
	}

	for(i=0;i<3;i++){
		num[i] = i;
		err[i] = pthread_create(&barber[i],NULL,Barber,&num[i]);
		if(err[i] != 0){
			printf("can't create thread: %s\n",strerror(err[i]));
		}
	}

	while(!getchar());
	return 0;
}
```
