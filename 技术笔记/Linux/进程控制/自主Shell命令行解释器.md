## 第一步：通过虚拟环境获取必要信息

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

// 分别从虚拟环境中获取用户、主机、路径
const char *GetUserName()
{
    const char *name = getenv("USER");
    return name == NULL ? "None" : name;
}

const char *GetHostName()
{
    const char *hostname = getenv("HOSTNAME");
    return hostname == NULL ? "None" : hostname;
}

const char *GetPwd()
{
    const char *pwd = getenv("PWD");
    return pwd == NULL ? "None" : pwd;
}

int main()
{
    printf("[%s@%s %s]#", GetUserName(), GetHostName(), GetPwd());

    return 0;
}
```

## 第二步：等待用户输入

需要等待用户进行输入，可利用fget()函数来实现
fget()函数的优点：
1. 可以字符串的方式输入
2. 可等待用户输入
3. 可利用函数清理换行
```c
#include <iostream>
#include <cstdio>
#include <cstring>
#include <cstdlib>

#define COMMAND_SIZE 1024

// 分别从虚拟环境中获取用户、主机、路径
const char *GetUserName()
{
    const char *name = getenv("USER");
    return name == NULL ? "None" : name;
}

const char *GetHostName()
{
    const char *hostname = getenv("HOSTNAME");
    return hostname == NULL ? "None" : hostname;
}

const char *GetPwd()
{
    const char *pwd = getenv("PWD");
    return pwd == NULL ? "None" : pwd;
}

int main()
{
    printf("[%s@%s %s]# ", GetUserName(), GetHostName(), GetPwd());

    // ls -a -l => "ls -a -l" 字符串
    char commandline[COMMAND_SIZE];
    char *c = fgets(commandline, sizeof(commandline), stdin);
    if(c == NULL) return 1; /*失败返回 1*/
    commandline[strlen(commandline)-1] = 0; // 清理\n
    printf("echo %s\n", commandline);

    return 0;
}
```

## 面向对象封装风格

```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <cstdlib>

#define COMMAND_SIZE 1024
#define FORMAT "[%s@%s %s]# "

// 分别从虚拟环境中获取用户、主机、路径
const char *GetUserName()
{
    const char *name = getenv("USER");
    return name == NULL ? "None" : name;
}

const char *GetHostName()
{
    const char *hostname = getenv("HOSTNAME");
    return hostname == NULL ? "None" : hostname;
}

const char *GetPwd()
{
    const char *pwd = getenv("PWD");
    return pwd == NULL ? "None" : pwd;
}

// 制作输出命令
void MakeCommandline(char cmd_prompt[], int size)
{
    snprintf(cmd_prompt, size, FORMAT, GetUserName(), GetHostName(), GetPwd());
}

void PrintCommandPrompt()
{
    char prompt[COMMAND_SIZE];
    MakeCommandline(prompt, sizeof(prompt)); // 制作
    printf("%s", prompt); // 输出
    fflush(stdout);
}

bool GetCommandLine(char *out, int size)
{
    // ls -a -l => "ls -a -l" 字符串
    char *c = fgets(out, size, stdin);
    if(c == NULL) return false;
    out[strlen(out)-1] = 0; // 清理\n
    if(strlen(out) == 0) return false; // 用户只回车
    return true;
}

int main()
{
    // 1. 输出命令行提示符
    PrintCommandPrompt();

    // 2. 获取用户输入的命令
    char commandline[COMMAND_SIZE];
    if(GetCommandLine(commandline, sizeof(commandline)))
    {
        printf("echo %s\n", commandline); // 输入成功，回显命令
    }

    return 0;
}
```