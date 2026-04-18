# 第一步：通过虚拟环境获取必要信息

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

# 第二步：等待并获取命令字符串

## 字符串的读写

使用C语言中的`scanf()` 和 C++中的 `cin<<`，会省略空格
比如输入 `ls -l -a` -> `ls-l-a`

如要解决，可选择用 `fgets()` 函数，
fget()函数的优点：
1. 可以字符串的方式输入
2. 可等待用户输入
3. 可利用函数清理换行

```cpp
#define COMMAND_SIZE 1024
#define MAXARGC 128
char *g_argv[MAXARGC];
int g_argc = 0;

// 获取用户输入的命令
bool GetCommandLine(char *out, int size)
{
    // ls -a -l => "ls -a -l" 字符串
    char *c = fgets(out, size, stdin); // stdin输入到out
    if(c == NULL) return false;
    
    if(strlen(out) == 0) return false; // 用户输入回车
    return true;
}

int main()
{
    while(true)
    {
        // 2. 获取用户输入的命令
        char commandline[COMMAND_SIZE];
        if(!GetCommandLine(commandline, sizeof(commandline)))
            continue;
    }
    return 0;
}
```

## 清理字符串结尾的 \n

成功读写字符串，尾部会自动加上`\n`。当我们回车，字符串尾部的`\n` 与 `回车`形成两个回车
```Linux
[用户]# ls -l -a

[用户]#
```

可将 out 的长度往左移一位，将`\n`省去
```cpp
out[strlen(out)-1] = 0; // 清理\n
```

## 完整代码

```cpp
#define COMMAND_SIZE 1024
#define MAXARGC 128
char *g_argv[MAXARGC];
int g_argc = 0;

// 获取用户输入的命令
bool GetCommandLine(char *out, int size)
{
    // ls -a -l => "ls -a -l" 字符串
    char *c = fgets(out, size, stdin); // stdin输入到out
    if(c == NULL) return false;
	out[strlen(out)-1] = 0; // 清理\n
    if(strlen(out) == 0) return false; // 用户输入回车
    return true;
}

int main()
{
    while(true)
    {
        // 2. 获取用户输入的命令
        char commandline[COMMAND_SIZE];
        if(!GetCommandLine(commandline, sizeof(commandline)))
            continue;
    }
    return 0;
}
```

# 面向对象封装风格

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

# 第三步：拆解字符串命令（为下一步的子进程做准备）

## 用 strtok() 函数对字符串进行分割
```cpp
char * strtok ( char * str, const char * delimiters );
```
- `If a token is found, a pointer to the beginning of the token. ` 
- `Otherwise, a _null pointer_.`

```cpp
// 将字符串"ls -a -l" 拆成 "ls" "-a" "-l"
bool CommandParse(char *commandline) // 传入的字符串
{
#define SEP " "
    g_argc = 0; // 保证每次 g_argc = 0
    // 命令行分析"ls -a -l" 拆成 "ls" "-a" "-l"
    g_argv[g_argc++] = strtok(commandline, SEP);

    // 循环切完命令的每一个参数，填入g_argv里
    while((bool)(g_argv[g_argc++] = strtok(nullptr, SEP)));

    return true;
}
```

## g_argc 个数

`g_argv[g_argc++]` 为后置++，大小多一个，需 `g_argc--;`
```cpp
g_argc--;
```

## 完整代码
```cpp
// 3. 将字符串"ls -a -l" 拆成 "ls" "-a" "-l"
bool CommandParse(char *commandline)
{
#define SEP " "
    g_argc = 0; // 保证每次 g_argc = 0
    // 命令行分析"ls -a -l" 拆成 "ls" "-a" "-l"
    g_argv[g_argc++] = strtok(commandline, SEP);

    // 循环切完命令的每一个参数，填入g_argv里
    while((bool)(g_argv[g_argc++] = strtok(nullptr, SEP)));

    // 因为g_argc为后置++，把尾部的NULL也带上了，需要减去一个
    g_argc--;
    return true;
}

// 检查切割是否正确
void PrintArgv()
{
    for(int i = 0; g_argv[i]; i++)
    {
        printf("argv[%d]->%s\n", i, g_argv[i]);
    }
    printf("argc: %d\n", g_argc);
}

int main()
{
    while(true)
    {
        // 3. 将字符串"ls -a -l" 拆成 "ls" "-a" "-l"
        CommandParse(commandline); 
        //PrintArgv();
    }
    return 0;
}

```

# 第四步：执行命令

需要创建子进程来执行命令，需用到 `execl` 接口
```cpp
#include <sys/types.h>
#include <sys/wait.h>

#define COMMAND_SIZE 1024
#define FORMAT "[%s@%s %s]# "

// 下面是shell定义的全局数据
#define MAXARGC 128
char *g_argv[MAXARGC];
int g_argc = 0;

int Execute()
{
    pid_t id = fork(); // 创建子进程
    if(id == 0)
    {
        // child
        execvp(g_argv[0], g_argv); // 执行命令
        exit(1);
    }

    // father   
    pid_t rid = waitpid(id, nullptr, 0); // 子进程结束，子进程变成僵尸进程，需要被父进程接受
    (void)rid; // rid 使用一下
    return 0;
}

int main()
{
    while(true)
    {
        // 4. 执行命令
        Execute();
    }
    return 0;
}
```

# 第五步：内建命令的检查

子进程执行 `cd` 命令，改变的是子进程的路径，子进程会被销毁掉，对 Bush 没有影响。想改变当前路径，需父进程执行 `cd` 命令，也就是内建命令

## cd 命令

改变路径时，需要用到 chdir() 函数来改变路径
```cpp
NAME
       chdir, fchdir - change working directory

SYNOPSIS
       #include <unistd.h>

       int chdir(const char *path);
```

1. 单独只有 cd 时，直接回到家目录
2. cd 有参数时，进入到参数路径

```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

#define COMMAND_SIZE 1024
#define FORMAT "[%s@%s %s]# "

// 下面是shell定义的全局数据
#define MAXARGC 128
char *g_argv[MAXARGC];
int g_argc = 0;


// 检查是否是内建命令
// 不需要传参数，虚拟地址为全局变量
bool CheckAndExecBuiltin()
{
    std::string cmd = g_argv[0];
    if(cmd == "cd")
    {
        return true;
    }

    return false;
}

int Execute()
{
    pid_t id = fork();
    if(id == 0)
    {
        // child
        execvp(g_argv[0], g_argv); 
        exit(1);
    }

    // father   
    pid_t rid = waitpid(id, nullptr, 0);
    (void)rid; // rid 使用一下
    return 0;
}

int main()
{
    while(true)
    {
        // 4. 检查并处理内建命令
        if(CheckAndExecBuiltin()) // 1.内建命令 2.非内建命令
            continue;
        
        // 5. 执行命令
        Execute();
    }
    return 0;
}
```

# 第六步：进程虚拟环境不改变，就用系统路径来改变

路径变化时，由于进程的虚拟环境没有刷新，环境变量一直都是老环境
如果想改变路径，用系统路径获取最新路径
```cpp
NAME
       getcwd, getwd, get_current_dir_name - get current working directory

SYNOPSIS
       #include <unistd.h>

       char *getcwd(char *buf, size_t size);

       char *getwd(char *buf);
```

```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

#define COMMAND_SIZE 1024
#define FORMAT "[%s@%s %s]# "

// 下面是shell定义的全局数据
#define MAXARGC 128
char *g_argv[MAXARGC];
int g_argc = 0;

// 单独定义环境变量，需更新环境变量
char cwd[1024];
char cwdenv[1024];

const char *GetPwd()
{
    //const char *pwd = getenv("PWD");
    
    // 利用getcwd虚拟调用，获取最新路径
    const char *pwd = getcwd(cwd, sizeof(cwd));
    if(pwd != NULL)
    {
        snprintf(cwdenv, sizeof(cwdenv), "PWD=%s", cwd); // 格式化
        putenv(cwdenv); // 导出环境变量，更新环境变量
    }
    return pwd == NULL ? "None" : pwd;
}

const char *GetHome()
{
    const char *home = getenv("HOME");
    return home == NULL ? "None" : home;
}

// 检查是否是内建命令
// 不需要传参数，虚拟地址为全局变量
bool CheckAndExecBuiltin()
{
    std::string cmd = g_argv[0];
    if(cmd == "cd")
    {
        // 1. 只有cd argc=1  2. 带参数 argc=2
        if(g_argc == 1)
        {

            std::string home = GetHome();
            if(home.empty()) return true;
            chdir(home.c_str()); // 切换到家路径
        }
        else
        {
            std::string where = g_argv[1];
            // cd - / cd ~
            if(where == "-")
            {
                // ...
            }
            else if(where == "~")
            {
                // ...
            }
            else
            {
                // 切换到参数路径
                chdir(where.c_str());
            }
        }
        return true;
    }

    return false;
}

int main()
{
    while(true)
    {
        // 3. 将字符串"ls -a -l" 拆成 "ls" "-a" "-l"
        if(!CommandParse(commandline))
            continue;
        //PrintArgv();
        
        // 4. 检查并处理内建命令
        if(CheckAndExecBuiltin()) // 1.内建命令 2.非内建命令
            continue;
    }
    return 0;
}
```

```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

#define COMMAND_SIZE 1024
#define FORMAT "[%s@%s %s]# "

// 下面是shell定义的全局数据
#define MAXARGC 128
char *g_argv[MAXARGC];
int g_argc = 0;

// 单独定义环境变量，需更新环境变量
char cwd[1024];
char cwdenv[1024];

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
    //const char *pwd = getenv("PWD");
    
    // 利用getcwd虚拟调用，获取最新路径
    const char *pwd = getcwd(cwd, sizeof(cwd));
    if(pwd != NULL)
    {
        snprintf(cwdenv, sizeof(cwdenv), "PWD=%s", cwd); // 格式化
        putenv(cwdenv); // 导出环境变量，更新环境变量
    }
    return pwd == NULL ? "None" : pwd;
}

const char *GetHome()
{
    const char *home = getenv("HOME");
    return home == NULL ? "None" : home;
}

// 路径处理
std::string DirName(const char *pwd)
{
#define SLASH "/"
    std::string dir = pwd;
    if(dir == SLASH) return SLASH;
    auto pos = dir.rfind(SLASH); // 反向查找 "/"
    if(pos == std::string::npos) return "BUG!";
    return dir.substr(pos+1); // 省略"/"
}

// 制作输出命令
void MakeCommandline(char cmd_prompt[], int size)
{
    // snprintf(cmd_prompt, size, FORMAT, GetUserName(), GetHostName(), DirName(GetPwd()).c_str());
    snprintf(cmd_prompt, size, FORMAT, GetUserName(), GetHostName(), GetPwd());
}

// 输出命令
void PrintCommandPrompt()
{
    char prompt[COMMAND_SIZE];
    MakeCommandline(prompt, sizeof(prompt)); // 制作
    printf("%s", prompt); // 输出
    fflush(stdout);
}

// 获取用户输入的命令
bool GetCommandLine(char *out, int size)
{
    // ls -a -l => "ls -a -l" 字符串
    char *c = fgets(out, size, stdin);
    if(c == NULL) return false;
    out[strlen(out)-1] = 0; // 清理\n
    if(strlen(out) == 0) return false; // 用户输入回车
    return true;
}

// 3. 将字符串"ls -a -l" 拆成 "ls" "-a" "-l"
bool CommandParse(char *commandline)
{
#define SEP " "
    g_argc = 0; // 保证每次 g_argc = 0
    // 命令行分析"ls -a -l" 拆成 "ls" "-a" "-l"
    g_argv[g_argc++] = strtok(commandline, SEP);

    // 循环切完命令的每一个参数，填入g_argv里
    while((bool)(g_argv[g_argc++] = strtok(nullptr, SEP)));

    // 因为g_argc为后置++，把尾部的NULL也带上了，需要减去一个
    g_argc--;
    return g_argc > 0 ? true : false;
}

void PrintArgv()
{
    for(int i = 0; g_argv[i]; i++)
    {
        printf("argv[%d]->%s\n", i, g_argv[i]);
    }
    printf("argc: %d\n", g_argc);
}

// 检查是否是内建命令
// 不需要传参数，虚拟地址为全局变量
bool CheckAndExecBuiltin()
{
    std::string cmd = g_argv[0];
    if(cmd == "cd")
    {
        // 1. 只有cd argc=1  2. 带参数 argc=2
        if(g_argc == 1)
        {

            std::string home = GetHome();
            if(home.empty()) return true;
            chdir(home.c_str()); // 切换到家路径
        }
        else
        {
            std::string where = g_argv[1];
            // cd - / cd ~
            if(where == "-")
            {
                // ...
            }
            else if(where == "~")
            {
                // ...
            }
            else
            {
                // 切换到参数路径
                chdir(where.c_str());
            }
        }
        return true;
    }

    return false;
}

int Execute()
{
    pid_t id = fork();
    if(id == 0)
    {
        // child
        execvp(g_argv[0], g_argv); 
        exit(1);
    }

    // father   
    pid_t rid = waitpid(id, nullptr, 0);
    (void)rid; // rid 使用一下
    return 0;
}

int main()
{
    while(true)
    {
        // 1. 输出命令行提示符
        PrintCommandPrompt();

        // 2. 获取用户输入的命令
        char commandline[COMMAND_SIZE];
        if(!GetCommandLine(commandline, sizeof(commandline)))
            continue;

        // 3. 将字符串"ls -a -l" 拆成 "ls" "-a" "-l"
        if(!CommandParse(commandline))
            continue;
        //PrintArgv();
        
        // 4. 检查并处理内建命令
        if(CheckAndExecBuiltin()) // 1.内建命令 2.非内建命令
            continue;
        
        // 5. 执行命令
        Execute();
    }
    return 0;
}
```