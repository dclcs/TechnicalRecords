## Week 02 Segment2: open(2) / close(2)


- almost all file I/O can be performed using just these five functions:
  - open(2) and close(2)
  - read(2), write(2), 
  - and lseek(2),




### create a file


- The creat(2) system call. 
  - This call takes a pathname as the first argument 
  - and a mode_t - a description of the access permissions -- as the second argument, 
  - and it will dutifully create the new file, open it in write mode, and return to you the file descriptor representing this new file.

```c
#include<fcntl.h>

int creat(const char *pathname, mode_t mode)
// Returns: file descriptor if OK, -1 on error
```



**PS:**
```text
creat(2) returns to you a file descriptor opened in write-only mode, but sometimes you want to create a new file and get back a file handle that allows both reading and writing.

In the early days of Unix, when open(2) wasn't able to create files, you had to creat(2) the file, close(2) it, then open(2) it again.

This posed a race condition, whereby another process could have, for example, unlinked the file and put in place a different file, leading to unexpected results.

That is, the creation and opening of the file was not an `_atomic_` operation. An atomic operation is one where all steps are guaranteed to complete without the possibility for another process to change state during the operation.


So the creat(2) call was obsoleted by open(2) with the flags used as shown here.  Which gets us to the open(2) system call...
```


### OPEN System call
```c
#include<fcntl.h>
int open(const char* pathname, int oflag, ... /* mode_t mode*/);
// Returns: file descriptor if OK, -1 On error
```
- The open(2) syscall 
  - argument a pathname, 
  - a bitmask of flags telling open(2) how to behave, 
  -  optionally, as a third argument the 'mode' permissions with which a file will be created, subject to modifications by the process's umask(2), 


- Note that this makes open(2) the only syscall we discuss here that doesn't take a file descriptor  which makes sense, since it's the one call that _returns_ the file descriptor.

- let's take a look at the 'oflags':
  - O_RDONLY - open for reading only
  - O_WROBLY - open for writing only
  - O_RDWR - open for reading and writing


- Then there are a few additional flags you can enable by ORing the flags.  They are:

  - O_APPEND, which we'll see in practice in a little bit

  - O_CREAT, which is what allowed us to obsolete the creat(2) syscall; if O_CREAT is specified, then the  third argument 'mode' is required

  - O_EXCL allows for an atomic exclusive opening of a file, since without this a test for the existence of the file followed by the call to open it, therebyagain introducing a race condition.This is actually a common vulnerability pattern known as TOCTOU - a time-of-check versus time-of-use race condition, so O_EXCL avoids this problem.

  - O_TRUNC truncates the file



- Note also that the flags supported by open(2) may differ across platforms and go beyond what POSIX requires.  


Now open(2) will return to us a file descriptor on success, but of course things don't always go well.When open(2) fails, it will return -1 and set errno appropriately.

Some of the most common errors why open(2) may fail are shown here:
- EEXIST: O_CRREAT|O_EXCL was specified but the file exists
- EMFILE: process has already reached max numbed of open file descriptors
- EMOENT: file does not exist
- EPERM: lack of permissions


There are many, many other scenarios under which
open(2) may fail - check your manual page!


### OPENAT System Call
- Finally, many Unix versions nowadays also support the openat(2) system call, which is used to handle relative pathnames from a different working directory in an atomic fashion.


```c
#include<fcntl.h>
int open(const char* pathname, int oflag, ... /* mode_t mode*/);
int openat(int dirfd, const char* pathname, int oflag, .../*mode_t mode*/);
// Returns: file descriptor if OK, -1 On error
```

- consider that you can change your working directory and may wish to resolve a pathname relative to wherever you started without having to either change back or to have to construct an absolute path.

- Perhaps consider this another exercise for you: think of a scenario where openat(2) can help prevent atime-of-check/time-of-use race condition,



### Close System call

- Closing a file is a bit easier.  You pass in the file descriptor, and that's about it.

```c
#include<fcntl.h>
int close(int fd);
// Returns: 0 if OK, -1 On error
```

Can close(2) not fail?

If you look at the manual page, you'll find that close(2) _can_ fail: it can fail if the number ou
gave it was not a file descriptor, or if it received a hardware interrupt, but that's about it.

What do you do if your close(2) call fails?  Well, you were going to close the file descriptor, so you wouldn't be using it after the close(2) call anyway, so in most cases it is actually permissible to move on even if close(2) failed.

However, as careful programmers, we want to make sure that the reader understands that we are not blindly ignoring the return value, so we explicitly cast it to void.

We'll note that closing a file descriptor will release any record locks on that file; .

Now in simple programs, you can even get away with not calling close(2) yourself at all -- when your process exits, the kernel will automatically close any openfile descriptors you had for you.

But that is 
- (a) sloppy programming, 
- and (b) easily leads to you leaking file descriptors when you refactor your code, and all of a sudden your open(2) call is made inside a loop.

To prevent this, you should get into the habit of
always closing your file descriptors explicitly and within the same scope of your open(2) call.

Example:

```c
void func() {
    int fd;
    if((fd = open(path, O_RDONLY)) < 0 ) {
        perror("unable to open");
        exit(EXIT_FAILURE);
    }

    /* ok do something... */
    (void) close(fd);
}
int main() {
    
    func()
    /* imagine a few dozen lines of code there */
}
```