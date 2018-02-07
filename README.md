One of the first _public_ LKM (loadable kernel module) rootkits; posted to Bugtraq in '97 by Runar Jensen
http://seclists.org/bugtraq/1997/Oct/55
```
From: zarq () 1STNET COM (Runar Jensen)
Date: Thu, 9 Oct 1997 00:37:52 -0500

As halflife demonstrated in Phrack 50 with his linspy project, it is trivial
to patch any system call under Linux from within a module. This means that
once your system has been compromised at the root level, it is possible for
an intruder to hide completely _without_ modifying any binaries or leaving
any visible backdoors behind. Because such tools are likely to be in use
within the hacker community already, I decided to publish a piece of code to
demonstrate the potentials of a malicious module.

The following piece of code is a fully working Linux module for 2.1 kernels
that patches the getdents(), kill(), read() and query_module() calls. Once
loaded, the module becomes invisible to lsmod and a dump of /proc/modules by
modifying the output of every query_module() call and every read() call
accessing /proc/modules. Apparently rmmod also calls query_module() to list
all modules before attempting to remove the specified module, and will
therefore claim that the module does not exist even if you know its name. The
output of any getdents() call is modified to hide any files or directories
starting with a given string, leaving them accessible only if you know their
exact names. It also hides any directories in /proc matching pids that have a
specified flag set in its internal task structure, allowing a user with root
access to hide any process (and its children, since the task structure is
duplicated when the process does a fork()). To set this flag, simply send the
process a signal 31 which is caught and handled by the patched kill() call.

To demonstrate the effects...

[root@image:~/test]# ls -l
total 3
-rw-------   1 root     root         2832 Oct  8 16:52 heroin.o
[root@image:~/test]# insmod heroin.o
[root@image:~/test]# lsmod | grep heroin
[root@image:~/test]# grep heroin /proc/modules
[root@image:~/test]# rmmod heroin
rmmod: module heroin not loaded
[root@image:~/test]# ls -l
total 0
[root@image:~/test]# echo "I'm invisible" > heroin_test
[root@image:~/test]# ls -l
total 0
[root@image:~/test]# cat heroin_test
I'm invisible
[root@image:~/test]# ps -aux | grep gpm
root       223  0.0  1.0   932   312  ?  S   16:08   0:00 gpm
[root@image:~/test]# kill -31 223
[root@image:~/test]# ps -aux | grep gpm
[root@image:~/test]# ps -aux 223
USER       PID %CPU %MEM  SIZE   RSS TTY STAT START   TIME COMMAND
root       223  0.0  1.0   932   312  ?  S   16:08   0:00 gpm
[root@image:~/test]# ls -l /proc | grep 223
[root@image:~/test]# ls -l /proc/223
total 0
-r--r--r--   1 root     root            0 Oct  8 16:53 cmdline
lrwx------   1 root     root            0 Oct  8 16:54 cwd -> /var/run
-r--------   1 root     root            0 Oct  8 16:54 environ
lrwx------   1 root     root            0 Oct  8 16:54 exe -> /usr/bin/gpm
dr-x------   1 root     root            0 Oct  8 16:54 fd
pr--r--r--   1 root     root            0 Oct  8 16:54 maps
-rw-------   1 root     root            0 Oct  8 16:54 mem
lrwx------   1 root     root            0 Oct  8 16:54 root -> /
-r--r--r--   1 root     root            0 Oct  8 16:53 stat
-r--r--r--   1 root     root            0 Oct  8 16:54 statm
-r--r--r--   1 root     root            0 Oct  8 16:54 status
[root@image:~/test]#

The implications should be obvious. Once a compromise has taken place,
nothing can be trusted, the operating system included. A module such as this
could be placed in /lib/modules/<kernel_ver>/default to force it to be loaded
after every reboot, or put in place of a commonly used module and in turn
have it load the required module for an added level of protection. (Thanks
Sean:) Combined with a reasonably obscure remote backdoor it could remain
undetected for long periods of time unless the system administrator knows
what to look for. It could even hide the packets going to and from this
backdoor from the kernel itself to prevent a local packet sniffer from seeing
them.

So how can it be detected? In this case, since the number of processes is
limited, one could try to open every possible process directory in /proc and
look for the ones that do not show up otherwise. Using readdir() instead of
getdents() will not work, since it appears to be just a wrapper for
getdents(). In short, trying to locate something like this without knowing
exactly what to look for is rather futile if done in userspace...

Be afraid. Be very afraid. ;)


.../ru
```
