
进程	元数据（pid，状态，文件，优先级...）
文件	元数据(大小，权限，日期。。。)

OS
	经常我们会产生新的子进程
	子进程不会运行原来父进程的代码，而是要执行新的程序
	
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <unistd.h>   
#define SIZE 80

int main(int argc, char const *argv[]){
    printf("main:pid=%d\n", getpid());
    char *cmd=malloc(SIZE);
    for (int i=1;i<argc;i++){
        strncat(cmd,argv[i],strlen(argv[i]));
        strncat(cmd," ",1);
    }
    printf("shell command:%s\n",cmd);
    system(cmd);		//system("ps -ef");
    free(cmd);    return 0;
}
	
./a.out ps -ef   

	子进程执行新程序的方法
	1.system()
		产生子shell进程,并在其中执行新程序
	2.exec*()
		替换当前进程,并在其中执行新程序

#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main(int argc, char const *argv[]){
    printf("main_pid=%d\n",getpid());
    execl("/bin/ps","ps","a",NULL);
    return 0;
}


fork()
	通过其返回值去判断，是在子进程还是在父进程中执行
	
	
if(fork()==0){
	//子进程代码
}
else{
	//父进程代码
}
//父子共用代码



进程终止
	两种终止方式
		正常终止	exit()  _exit()
		异常终止	后续信号中讲解
	正常终止
		对exit()会执行一些基本关闭动作,然后通知内核终止
			调用退出处理程序 	atexit()注册
			刷新stdio流缓冲区
			用参数status(标识进程退出状态)执行_exit()系统调用
		main中return n 等同于exit(n)
		相对exit(),_exit()直接返回内核,简单粗暴     _exit(2)



#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

static void my_exit1(void){
    printf("1111 exit handler\n");
}
static void my_exit2(void){
    printf("2222 exit handler\n");
}
int main(void) {
    if(atexit(my_exit2) != 0){
        printf("register my_exit2 failed\n");
        return -1;
    }
    if(atexit(my_exit1) != 0){
        printf("register my_exit1 failed\n");
        return -1;
    }
    printf("main is done\n");
    return 0;  // exit(0);
}

	退出处理程序
	可通过注册在退出前做自定义清理的退出程序
	atexit()
		至少可注册32个
		先进后出  FILO


父进程可以监控子进程的退出状态
	如果子进程先于父进程退出
		父进程可通过wait()/waitpid()获得子进程退出状态
		如果父进程未及时wait(),子进程成为僵尸进程,直到父进程wait()回收才真正结束生命周期
			处于僵尸进程时已经释放了大部分资源但还保留内核进程表中的一条记录
			僵尸进程无法通过kill杀死		wait(2)/NOTES
	如果子进程后于父进程推出
		子进程成为孤儿进程,后续由init进程(pid=1)接管
		后续子进程结束时,init自动调用wait()移除僵尸进程



僵尸进程
#include <stdio.h>
#include <unistd.h>
int main(void){
  if(fork()==0){
    printf("child pid/ppid=%d:%d\n",getpid(),getppid());
  }else{
    sleep(1);
    getchar();  //回车前执行ps lf查看状态
  }
  return 0;
}	

子进程被init接管
#include <stdio.h>
#include <unistd.h>
int main(void){
  if(fork()==0){
    printf("child pid/ppid=%d:%d\n",getpid(),getppid());
    getchar();  //回车前执行ps lf查看状态
    printf("child pid/ppid=%d:%d\n",getpid(),getppid());
  }else
    sleep(1);
  return 0;
}

