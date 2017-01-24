---
layout: post
title : 套接字和管道示例
category : program
---
{% include JB/setup %}

这学期有门课程是TCP/IP网络套接字编程,虽然大一开始就接触Linux了,不过还没有编写过Linux程序,Java的套接字程序倒是写过.下面是Linux下套接字和管道的例子程序.

### 套接字

一个线程利用send函数发信息

一个线程利用recv接收信息并显示

```c
#include <stdio.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdlib.h>
#include <pthread.h>
#include <netdb.h>

#define SERVPORT 10000
#define BACKLOG 10 
#define MAXDATASIZE 100


void * Server(char *arg)
{
  int sockfd, client_fd, addr_size, recvbytes;
  char rcv_buf[MAXDATASIZE], snd_buf[MAXDATASIZE];
  char* val;
  struct sockaddr_in server_addr; 
  struct sockaddr_in client_addr;

  char IPdotdec[20];

  if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) == -1)
  {
    perror("socket:");
    return ((void *)1);
  }

  //服务端套接字设置
  server_addr.sin_family = AF_INET; 
  server_addr.sin_port = htons(SERVPORT); 
  server_addr.sin_addr.s_addr = INADDR_ANY;
  if (bind(sockfd, (struct sockaddr*)&server_addr, sizeof(struct sockaddr))== -1)
  {
    perror("bind:");
    return ((void *)1);
  }

  //开始监听
  if (listen(sockfd, BACKLOG) == -1)
  {
    perror("listen:");
    return ((void *)1);
  }

  addr_size = sizeof(struct sockaddr_in);

  //接受连接请求
	if ((client_fd = accept(sockfd, (struct sockaddr *)&client_addr, &addr_size)) == -1)
	{
		perror("accept:");
	  return ((void *)1);
	}

    
  inet_ntop(AF_INET, (void*)&client_addr, IPdotdec, 16);
  printf("connetion from:%d : %s\n",client_addr.sin_addr, IPdotdec);

  //开始循环接受消息
	while((recvbytes = recv(client_fd, rcv_buf, MAXDATASIZE, 0)) > 0)
	{
      rcv_buf[recvbytes]='\0';
      printf("recv:%s\n", rcv_buf);
	}

    close(client_fd);
  	return ((void *)0);
}


void * Client(char *arg)
{
  int sockfd, recvbytes;
  char rcv_buf[MAXDATASIZE];
  char snd_buf[MAXDATASIZE];
  struct hostent *host;             
  struct sockaddr_in server_addr;

  if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) == -1)
  {
    perror("socket:");
    return ((void *)1);
  }

  ////客户端套接字设置
  server_addr.sin_family = AF_INET;
  server_addr.sin_port = htons(SERVPORT);
  inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr);
  memset(&(server_addr.sin_zero), 0, 8);

  //创建到服务器的连接
  if (connect(sockfd, (struct sockaddr *)&server_addr, sizeof(struct sockaddr)) == -1)
  {
    perror("connect");
    return ((void *)1);
  }

  strcpy(snd_buf,"Test Case#:  ");
  int i=0;

  //开始循环发送消息
	for(i;i<10;i++)
  {
	snd_buf[12] = i + '0';
	printf("send:%s\n", snd_buf);
	if (send(sockfd, snd_buf, sizeof(snd_buf), 0) == -1)
	{
		perror("send:");
		return ((void *)1);
	}
	sleep(3);

  close(sockfd);
  return ((void *)0);
}


int main(void)
{
   pthread_t ser;
   pthread_t cli;
   //先创建一个服务端线程再创建一个客户端进程
   pthread_create(&ser,NULL,(void *)Server,NULL);
   sleep(3);
   pthread_create(&cli,NULL,(void *)Client,NULL);
   while(!getchar());
   return 0;
}
```

运行截图:

<center><img alt="socket-and-pipe-1" src="{{ ASSET_PATH }}hooligan/img/post/socket-and-pipe-1.PNG"/></center>

### 管道

父进程创建一个子进程

父进程负责读用户终端输入，并写入管道

子进程从管道接收字符流写入另一个文件


```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int pipe_default[2];  

int main()
{
    pid_t pid;
    char buffer[32];
    memset(buffer, 0, 32);

    //创建管道
    if(pipe(pipe_default) < 0)
    {
        printf("Failed to create pipe!\n");
        return 0;
    }

    //创建子进程
    if(0 == (pid = fork()))
    {
        //将管道中读入的内容写入文件
		FILE *f = fopen("pipe.txt","wb");
        close(pipe_default[1]);
		int i=0;
		for(i;i<10;i++){
		    if(read(pipe_default[0], buffer, 32) > 0)
		    {
				fputs(buffer,f);
		    }
		}
		fclose(f);
        close(pipe_default[0]);
    }
    else
    {
        //接受用户的输入,并写入管道
        close(pipe_default[0]);
		char str[20];
		int i=0;
		for(i;i<10;i++)
		{
			printf("Please input a string:");
			fgets(str,19,stdin);
		    write(pipe_default[1], str, 20);

		}
        close(pipe_default[1]);
        waitpid(pid, NULL, 0);
    }

    return 1;
}
```

运行截图:

<center><img alt="socket-and-pipe-2" src="{{ ASSET_PATH }}hooligan/img/post/socket-and-pipe-2.PNG"/></center>

<center><img alt="socket-and-pipe-3" src="{{ ASSET_PATH }}hooligan/img/post/socket-and-pipe-3.PNG"/></center>
