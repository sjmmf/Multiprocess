```c
#include<unistd.h>
#include<sys/types.h>
#include<stdio.h>
#include<errno.h>

const int MAX = 100;      //数字上限
const int READ = 0;       //用于read函数的参数
const int WRITE = 1;      //用于write函数的参数

/*此函数用来判断是否是素数*/
void primes(int pipe_read)
{
	int prime;
	if (read(pipe_read, &prime, sizeof(int)) <= 0) {
		exit(0);
	}
	printf("prime %d\n", prime);

	int fd[2];     //声明一个管道
	pipe(fd);

	if (fork() == 0)
	{
		close(fd[WRITE]); //子进程关闭写端
		primes(fd[READ]);
		close(fd[READ]);  //子进程关闭读端
	}
	else 
	{
		int temp;
		close(fd[READ]);  //父进程关闭读端
		while (read(pipe_read, &temp, sizeof(int))) {
			if (temp % prime != 0) {
				write(fd[WRITE], &temp, sizeof(int));  //不是当前素数的倍数的数写入管道中
			}    
		}
		close(fd[WRITE]);  //父进程关闭写端
		wait(0);
	}
	exit(0);
}
int main()
{
	int fd[2];
	pipe(fd);
	int pid = fork();
	if (pid == 0)
	{
		close(fd[WRITE]);   //子进程关闭写端
		primes(fd[READ]);
		close(fd[READ]);    //子进程关闭读端
	}
	else 
	{
		close(fd[READ]);    //父进程关闭读端
		int i;
		for (i = 2; i <= MAX; i++) {
			write(fd[WRITE], &i, sizeof(int));
		}
		close(fd[WRITE]);   //父进程关闭写端
		wait(0);
	}
	exit(0);
}
```

![image-20220812192404229](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220812192404229.png)