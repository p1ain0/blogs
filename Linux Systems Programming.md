# Linux Systems Programming

## low level I/O

Unbuffered I/O
Sequential access
Random access

Using the standard library
Buffered I/O
Formatted I/O

Advanced Techniques
Scatter/gather I/O
Mapping files into memory

Demonstration:
Four ways to copy a file

### Unbuffered I/O

#### Open a file

```c
fd = open(name, flags, modes);
//name  : pathname of the file to be open
//flags : Must include one of : O_RDONLY O_WRONLY O_RDWR; May inlcude one or more of : O_APPEND O_CREAT O_TRUNC
//modes : Access mode of the file (Only used if the file is being created)
//fd    : Return lowest available descriptor
```

                fd=0               fd = 1,stdout
                stdin             ------------>
    Keyboard ----------> program                    screen(tty)
                                  ------------>
                                   fd = 2, stderr

#### Unbuffered Output

```c
write(fd, buffer, count);
```

#### Unbuffered Input

```c
read(fd, buffer, count);
//Return the number of bytes actually read
```

#### Close a file

```c
close(fd);
//Closes the file descriptor Makes it avaliable for re-use
//** There is a finite limit on how many descriptors a process can have open
```

#### demo : file copy

```c++
#include <fcntl.h>
#include <stdlib.h>
#define BSIZE 16384

int main()
{
    int fin, fout;
    char buf[BSIZE];
    int count;
    if((fin = open("foo", O_RDONLY)) < 0)
    {
        perror("foo");
        exit(errno);
    }
    if((fout = open("bar", O_WRONLY | O_CREAT, 0644)) < 0)
    {
        perror("bar");
        exit(errno);
    }
    while((count = read(fin, buf, BSIZE)) > 0)
        write(fout, buf, count);
    close(fin);
    close(fout);
    return 0;
}
```

#### Sequential Access

#### Random Access

The file positon pointer may be explicityly repositoned

```c
lseek(fd, offset, whence);
//fd : file descriptor
//offset : Byte offset. May be positive or negative
//whence : Specifies where the offset is relative to 
//SEEK_SET, Relative to start of file
//SEEK_CUR, Relative to current positive
//SEEK_END, Relative to end of file
```

#### Random Access Example

```c++
#include <unistd.h>
#include <fcntl.h>

struct record {
    int id;
    char name[80];
};

int main()
{
    int fd, size = sizeof(struct record);
    struct record info;
    fd = open("datafile", O_RDWR); //open for read write
    lseek(fd, size, SEEK_SET);  //skip one record
    read(fd, &info, size);  //read second record

    info.id = 99;           //Modify record
    lseek(fd, -size, SEEK_CUR); //Backspace
    write(fd, &info, size);     //Write modified record

    close(fd);

    return 0;
}
```

### Buffered and Formatted I/O

#### Open a file

```c
fd = fopen(name, mode);
//name : pathname of the file to be open
//mode : Valid modes include: "r", open text file for reading; "w", truncate and open for writing; "r+", open text file for update, Append "b" to the mode for binary file
//fd : Return a descriptor of type FILE*(or NULL on error)
```

#### Output

```c
fwrite(buffer, size, num, fd);
//Return the size of element actually write
```

#### Inpput

```c
fread(buffer, size, num, fd);
//Return the size of element actually read
```

#### Close a file

```c
fclose(fd);
```

### Difference of the two I/O

Feature | low level I/O | Standard Library I/O
---|---|---
Read/Write access | open() close() read() write() | fopen() fclose() fread() fwrite()
Random access | lseek() | fseek() rewind()
Type of descriptor | int | FILE*
User-space buffering | No | Yes
Part of C Library | No | Yes

### format I/O

```c
printf()
fprintf()
sprintf()
//%d decimal integer
//%8d right-justified in 8 character field
//%-8d left-justified in 8 character field
//%s  string
//%12.3f double, in 12 character field with 3 digits after the decimal point
```

#### demo : file copy2

```c++
#include <stdio.h>
#include <stdlib.h>
#define BSIZE 16384

int main()
{
    FILE* fin, fout;
    char buf[BSIZE];
    int count;
    if((fin = fopen("foo", "r")) == NULL)
    {
        perror("foo");
        exit(errno);
    }
    if((fout = fopen("bar", "w")) == NULL)
    {
        perror("bar");
        exit(errno);
    }
    while((count = fread(fin, buf, BSIZE)) > 0)
        fwrite(fout, buf, count);
    fclose(fin);
    fclose(fout);
    return 0;
}
```

#### scatter/gather IO

Read or Write multiple buffers of data in a singal call

Atomic

readv() and writev()

```c++
writev(fd, iov, iocouont);
```

#### Mapping files into Memory

```c
mmap(addr, length, prot, flags, fd, offset)
//addr: Set this to NULL to allow the kernel to choose the address
//length: The length of the mapping
//prot: PROT_READ PROT_WRITE
//flags: MAP_SHARED MAP_PRIVATE
//fd:file descriptor
//offset: offset within the file
//return the address at which the file has been mapped
```

example

```c++
#include <sys/mman.h>
#include <unistd.h>
#include <fcntl.h>

struct record {
    int id;
    char name[80];
};

int main()
{
    int fd, 
    size_t size;;
    struct record *info;
    fd = open("datafile", O_RDWR);
    size = lseek(fd, 0, SEEK_END);

    info = (struct record* )mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_PRIVATE, fd, 0);
    info[i].id = 99;
    msync(info, size, MS_SYNC);

    close(fd);
    return 0;
}
```

#### demo : file copy3

```c++
#include <unistd.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <string.h>

int main()
{
    char *src, *dst;
    int fin, fout;
    size_t size;

    fin = open("foo", O_REONLY);
    if(fin < 0)
    {
        perror("foo");
        exit(errno);
    }

    size = lseek(fin, 0, SEEK_END);

    src = mmap(NULL, size, PROT_READ, MAP_PRIVATE, fin, 0);
    if(src == MAP_FAILED)
    {
        perror("mmap");
        exit(errno);
    }

    fout = open("bar", O_RDWR | O_CREAT | O_TRUNC, S_IRUSR | S_IWUSR);
    if(fout < 0)
    {
        perror("bar");
        exit(errno);
    }
    if(ftruncate(fout, size) < 0)
    {
        perror("ftruncate");
        exit(errno);
    }

    dst = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_SHARED, fout, 0);
    if(dst == MAP_FAILED)
    {
        perror("mmap");
        exit(error);
    }
    memcpy(dst, src, size);
    if(msync(dst, size, MS_SYNC) < 0)
    {
        perror("msync");
        exit(error);
    }

    return 0;
}
```

## Manage File and Directories

File System Structure 
Links, nodes, directories

Working and Directories
Creating, deleting, listing

Advanced Techniques
Monitoring file events

Demonstraction:
Directory listing

### File System Structure

### Exam file Attribute

```c++
stat(name, buf);
//name : Pathname of the file to be exam
//buf : Pointer to a stat structure
fstat(fd, buf);
//fd : filedescriptorq
```

### File type and Permission

st_mode

| x | x | x | x | s | g | t | r | w | x | r | w | x | r | w | x |
| - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - |
| - | - | - | - | set uid | set gid | sticky bit| 
file type
010000 FIFO(pipe)
020000 character device
040000 directory
060000 block device
100000 regular file
120000 symbol link
140000 socket

Useful Macros:
S_ISREG
S_ISDIR
S_ISCHR
S_ISBLK
S_ISFIFO
S_ISLNK
S_ISSOCK

### Creating and Remove links

```c++
link(oldname, newname)
unlink(name)

```

#### Creating Symbol links

```c++
symlink(oldname, newname)
unlink(name)
```

#### difference

hard links: name ---> inode number
symbolic links: name ---> another name

### Interacting with directory

#### The Current Directory

every process has a "current Directory"

```c++
getcwd(buf, size);
```

#### Creating and removing Directory

```c++
mkdir(name, mode)
remove(name)
```

#### Reading Directory

```c++
d = opendir(dirname)
info = readdir(d)
closedir(d)
```

### Doing it in Python

function| Desctiptor|
--|--|
os.stat(path)| Return the stat_result instance similar to stat struct |
os.link(src, dst)| Create a new hard link |
os.symlink(src, dst)| Create a symbolic link |
os.chdir(path) | Change directory |
os.getcwd() | Get current work directory |
os.listdir(path) | opendir + readdir |

### Monitoring File System Event

#### File System Event

accessed modified deleted closed created moved

#### the inotify API

create a inotify instance

```c++
fd = inotify_init()
```

Adding to a inotity list

```c++
wd = inotify_add_watch(fd, path, mask)
```

Reading Events

Bit Value | Meaning |
-- | -- |
IN_ACCESS | File was accessed |
IN_ATTRIB | File attribute was attribute |
IN_CREAT | File created inside watched directory |
IN_DELETE | File deleted inside watched directory |
IN_DELETE_SELF | Watched file was delete |
IN_MODIFY | File was modified |
IN_MOVE_SELF | File was moved |

### the Command Line, the Environment, and Time

#### Accessing the command line

#### Processing the command options

```c++
getopt(argc, argv, optstring);

//optstring : strings specifying the allowed option letters. A following ":" says the option takes an argument.Example: "abc:d:"


while(a = getopt(argc, argv, "a") != EOF)
{
    switch(a){
        case 'a':
            c = optorg;
        default :
    }
}
```

#### The environment

a list of strings carried by each process

export XXX=xxx

```c++
getenv(name);
```

#### Time, Representions, conversions

Time zone and locale
Process times

Demonstrations

### Process and Pipe(Anonymous, named)

#### Process

fork()
exec() exec causes the current execution image of a process to be replaced by the image of new program

#### Pipe

named pipe : mkfiifo()

### User Accounts

#### Querying and Listing User Accounts

```c++
struct passwd{
    char *pw_name; //username
    char *pw_passwd; //user password
    char *pw_uid; //user ID
    char *pw_gid; //group ID
    char *pw_gecos; //user information
    char *pw_dir; //home directory
    char *pw_shell; //shell program 
};

getuid()
getpwnam()
getpwuid()
getpwent()
setpwent() //rewind to the beginning
getgid()
```

#### file permission

```c++
umask()
creat()
chmod()
```

#### file ownership

```c++
chown() //follow symbol link
lchown() //don't follow symbol link
```

## Mastering Sigals

### process terminal and exit status

```c++
wite(&status)
WIFEXITED(status)
WIFSIGNALED(status)
```

### establishing a signal handler

signal(SIGHUP, hup_handler)

### better signal handler with sigaction()

```c++
struct sigaction{
    void (*sa_handler)(int);
    sigset_t sa_mask;
    int sa_flags;
};

sigaddset()
sigdelset()
sigfillset()
sigemptyset()
sigismember()
sigprocmask()

```

### suggestions(seven things to do with signal)

1.Ignore them

```c++
signal(SIGINT, SIG_IGN);
or 
sigset_t s;
sigsfillset(&s);
sigdelset(&s, SIGHUP);
sigprocmask(SIG_SETMASK, &s, NULL);


```

2.Clean up and terminate

3.Reconfigure one on-the-fly

4.Report status dynamically

5.Turn debugging on and off

6.Implement a timeout

7.Schedule periodic actions