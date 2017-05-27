## 4.1 函数stat ##

函数stat返回与此命名文件有关的信息结构。下面的代码实现了一个工具，显示树形目录结构，需要加两个参数，一个为目录名，一个为显示目录的深度。
```
#include <sys/stat.h>
#include <sys/types.h>
#include <stdio.h>
#include <dirent.h>
#include <string.h>
#include <stdlib.h>
#include <fcntl.h>
#include <pwd.h>
#include <grp.h>
#include <time.h>
/***************************************************************/
 /*struct stat {*/
 /* unsigned long   st_dev; */    /* Device.  */
 /* unsigned long   st_ino;*/     /* File serial number.  */
 /* unsigned int    st_mode;  */  /* File mode.  */
 /* unsigned int    st_nlink;  */ /* Link count.  */
 /* unsigned int    st_uid;    */ /* User ID of the file's owner.  */
 /* unsigned int    st_gid;    */ /* Group ID of the file's group. */
 /* unsigned long   st_rdev;    *//* Device number, if device.  */
 /* unsigned long   __pad1;   */
 /* long        st_size;    *//* Size of file, in bytes.  */
 /* int     st_blksize; *//* Optimal block size for I/O.  */
 /* int     __pad2;         */
 /* long        st_blocks;*/  /* Number 512-byte blocks allocated. */
 /* long        st_atime;  */ /* Time of last access.  */
 /* unsigned long   st_atime_nsec; */
 /* long        st_mtime;  */ /* Time of last modification.  */
 /* unsigned long   st_mtime_nsec;*/
 /* long        st_ctime;  */ /* Time of last status change.  */
 /* unsigned long   st_ctime_nsec;  */
 /* unsigned int    __unused4;     */
 /* unsigned int    __unused5;*/
 /*                 };    */
 /* *************************************************************/

void printMode(unsigned int st_mode,int indent)
{
    int num = 0;
    for(;num<indent;num++)
    {
        printf("  ");
    }
    printf(S_ISDIR(st_mode)?"d":"-");
    printf(st_mode&S_IRUSR?"r":"-");
    printf(st_mode&S_IWUSR?"w":"-");
    printf(st_mode&S_IXUSR?"x":"-");
    printf(st_mode&S_IRGRP?"r":"-");
    printf(st_mode&S_IRGRP?"w":"-");
    printf(st_mode&S_IRGRP?"x":"-");
    printf(st_mode&S_IROTH?"r":"-");
    printf(st_mode&S_IROTH?"w":"-");
    printf(st_mode&S_IROTH?"x":"-");
}

void printFileName(char *name)
{
    printf(" %s\n",name);
}

void printUserName(unsigned int userId)
{
    struct passwd *pwd = getpwuid(userId);
    printf(" %s", pwd->pw_name);
}
void printGroupName(unsigned int grpId)
{
    struct  group *grp = getgrgid(grpId);
    printf(" %s" ,grp->gr_name);
}
void printSize(long size)
{
    printf(" %lu",size);
}
void printModifyTime(long mtime)
{
    /*char buf[100]={0};
    ctime_s(buf,26,mtime);
    printf(" %s",buf);*/
    printf(" %lu",mtime);
}
int ls(char *path,int depth,int indent)
{
    DIR *dhandle;
    struct dirent *file;
    struct stat st;
    if(!(dhandle=opendir(path)))
    {
        printf("error opendir %s\n",path);
        return -1;
    }
    while((file = readdir(dhandle))!=NULL)
    {
        int fullPathLen = strlen(path)+strlen(file->d_name)+1;
        if(strncmp(file->d_name,".",1)==0)
          continue;
        char *fullpath = (char*)malloc(fullPathLen+1);
        memset(fullpath,0,fullPathLen+1);
        strcpy(fullpath,path);
        strcat(fullpath,"/");
        strcat(fullpath,file->d_name);
        int rv = stat(fullpath,&st);
        if(rv<0)
        {
            return -1;
        }
        printMode(st.st_mode,indent);
        printUserName(st.st_uid);
        printGroupName(st.st_gid);
        printSize(st.st_size);
        printModifyTime(st.st_mtime);
        printFileName(file->d_name);

        if(S_ISDIR(st.st_mode)&& (depth-1>0))
        {
        ls(fullpath,depth-1,indent+1);
        }
        free(fullpath);
    }
    closedir(dhandle);
    return 0;
}

int main(int argc,char *argv[])
{
    int dep = atoi(argv[2]);
    ls(argv[1],dep,0);
    return 0;
}
```
运行如下命令

```
gcc stat.c
```
生成一个a.out可执行文件，运行如下命令：
```
harlan@DESKTOP-KU8C3K5:/github/APUE/chapter_4/myexamples$ ./a.out /github/ 2
drwxrwxrwx harlan harlan 0 1494143291 3202C
  drwxrwxrwx harlan harlan 0 1494143273 Doc
  -rw-rwxrwx harlan harlan 828 1494143273 Readme.txt
  drwxrwxrwx harlan harlan 0 1494143279 SRC
  drwxrwxrwx harlan harlan 0 1494143281 inc
  -rw-rwxrwx harlan harlan 9700 1494143282 m3327.mdf
  -rw-rwxrwx harlan harlan 1182 1494143282 m3327boot.mdf
  drwxrwxrwx harlan harlan 0 1494143290 prj
  drwxrwxrwx harlan harlan 0 1494143292 sdk
drwxrwxrwx harlan harlan 0 1495673220 APUE
  -rw-rwxrwx harlan harlan 6 1493303590 README.md
  -rwxrwxrwx root root 17478 1494424916 a.out
  -rw-rwxrwx harlan harlan 4352 1494167949 apue.h
  -rw-rwxrwx harlan harlan 2660400 1493735585 apue.h.gch
  drwxrwxrwx harlan harlan 0 1494248815 chapter_1
  drwxrwxrwx harlan harlan 0 1495117067 chapter_2
  drwxrwxrwx harlan harlan 0 1494509690 chapter_3
  drwxrwxrwx harlan harlan 0 1495113400 chapter_4
  drwxrwxrwx harlan harlan 0 1494944116 chapter_5
  -rw-rwxrwx harlan harlan 2220 1494167949 err.c
  drwxrwxrwx harlan harlan 0 1494769702 foo
  -rw-rwxrwx harlan harlan 399 1494424891 go.c
  -rw------- harlan harlan 1675 1494512317 key
  -rw-rwxrwx harlan harlan 404 1494512317 key.pub
  -rw-rwxrwx harlan harlan 1501 1494116048 print.c
  -rwx------ harlan harlan 1457 1493733958 tags
  drwxrwxrwx harlan harlan 0 1494769702 testdir
  -rw-rwxrwx harlan harlan 4790 1495671977 vimrc.txt
-rw------- harlan harlan 1679 1493304485 pub
-rw-rwxrwx harlan harlan 402 1493304485 pub.pub
drwxrwxrwx harlan harlan 0 1494511444 test
```

## 4.2 文件类型 ##

文件类型包括以下几种：
1. 普通文件
2. 目录文件
3. 块特殊文件
4. 字符特殊文件
5. FIFO
6. 套接字
7. 符号链接

可以用图4-1中的宏确定文件类型，这些宏的参数是stat结构中的st_mode成员。
![](http://files.cnblogs.com/files/harlanc/figure4-1.bmp)
可以用图4-2中的宏来从stat结构中确定IPC对象的类型。，它们的参数是指向stat结构的指针。

![](http://files.cnblogs.com/files/harlanc/figure4-2.bmp)

## 4.3 设置用户ID和设置组ID ##


![](http://files.cnblogs.com/files/harlanc/figure_4_5.bmp)

- 实际用户ID和实际组ID标识我们究竟是谁。
- 有效用户ID、有效组ID以及附属组ID决定了我们的文件访问权限。
- 保存的设置用户ID和保存的设置组ID在执行一个程序时包含了有效用户ID和有效组ID的副本。

通常，有效用户ID等于实际用户ID，有效组ID等于实际组ID。

我们可以在**文件模式字**(st_mode)中设置一个特殊标志，其含义是“当执行此文件时，将进程的有效用户ID设置为文件所有者的用户ID（st_uid）”,组ID也一样。在文件模式字中的这两位被称为**设置用户组ID（set-user=ID）位**和**设置组ID（set-group-ID）位**。

例如，passwd允许任意用户改变其口令，该程序是一个设置用户ID程序。见下面的展示：

```
harlan@DESKTOP-KU8C3K5:~$ ll /etc/passwd
-rw-r--r-- 1 root root 1219  5月  8 21:34 /etc/passwd
```
```
harlan@DESKTOP-KU8C3K5:~$ ll /usr/bin/passwd
-rwsr-xr-x 1 root root 47032  1月 27  2016 /usr/bin/passwd
```
任意用户修改口令需要写入/etc/passwd文件中，而此文件只有root用户才有写权限，/usr/bin/passwd用来执行写操作，可以看到这个程序将root用户指定为设置用户组ID，因此任意用户当执行此程序进行写文件的时候将拥有root权限。
## 4.5文件访问权限 ##
