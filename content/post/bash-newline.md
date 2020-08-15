---
title: "Bash Newline Magic"
date: 2020-08-14T21:42:54+05:30
Description: "Analysing bash weirdness with newline"
Tags: [bugbounty,penetration testing,bash,linux]
Categories: [bugbounty,penetration testing,bash,linux]
---

![main img](/img/main/bash-newline/main.png)

# init
Hi Y'ALL , this blog post covers on the analysis of weird thing in bash that is posted on [twitter](https://twitter.com/ajxchapman/status/1293900614724980743) by [Alex Chapman](https://twitter.com/ajxchapman). The image below shows the tweet . By creating files with weird names that have newline in the filename and creating a tar archive on these files and then executing using ./ (bash method) will be executed as a shell script. The irony here is bash will execute any files that has newline character in the starting bytes and we will see it in a while.

![tweet](https://pbs.twimg.com/media/EfTbgr8UcAcrhja?format=jpg&name=large)

# Normal Flow
First of all lets us try taring normal files instead of files with weird names.

![cmd img](/img/main/bash-newline/fileformaterr.png)

we use touch to create new empty files such as file1 and file2. we archive the files in tar format. To note, tar is not used for compression untill it is used with gzip which will end in .tar.gz . And then, chmod is used to give execution permission and then we execute it by using ./ . Bash tells us a sweet message 'exec file format error'. Ofcourse , we know that tar files are not executables. But bash rules the execution because it is the one that forks and exec the file and creates a child process(new process) and so bash will be the parent. Since bash is the parent of the commands that are run in bash terminal, we can use strace to see the workflow on how bash executes `./runme`. 

Now we will get the pid of the bash terminal from `ps aux` and then use strace to trace syscalls made by the process
![strace img](/img/main/bash-newline/normalflow.png)

![strace-1 img](/img/main/bash-newline/normalflow-strace.png)
Now we will run the tar file and analyse strace output. I have cut-shorted the strace output such that it contains only important information. As we see from the above image, it clones to create a new child process and executes the file runme by calling the exec syscall and it fails. The reason it fails is that the file should be of ELF format to get executed. The error message is returned from the bash, write syscall is called and finally it prints the error. Now it is all good, but we will try now the weird workflow.

## Weird Flow

Okay, now we insert newline character in starting bytes of the runme file . We just see hexdump and modify the starting bytes to include a newline using radare2.
![weird flow img](/img/main/bash-newline/weirdflow.png)

As we see from the above image it ran as a shell script. we will see the strace output to understand it better.
![weird strace output](/img/main/bash-newline/weirdflow-strace.png)

From the strace output,we can deduce that the bash will create a child process and executes `runme` using execve syscall, it failed as it is not a EFL format file. But since there is a newline, bash tries to execute it as a shell script.It tries to access /usr/bin/bash and it searches the file file1 (first file's filename in runme tar). since it can't find the file1 bin, it shows the message command not found. If we give valid command as a filename with newline in it, then it will be executed as shell script.

## exit
Now we will add a valid command with a newline in the filename and execute it. To include newline in the filename, we use the bash newline feature that is $'\n'. When bash sees this $'\n' it will be replaced by the newline. You can try it using `echo $'\n'`.

![final img](/img/main/bash-newline/newline-final.png)

In tar, filenames of the file included will be in the starting bytes and if we have newline in startingbytes, bash will execute its magic by executing the tar file as a shell script and it is not only the tar file can execute, any file can be executed if it contains newline in starting bytes. From the above image , newline is added in the filename by using $'\n' and when tared, the filenames will be in the starting bytes 690a(0a is the newline) and finally bash thinks that it is a shell script and executes. Another way of telling the bash to execute as a shell script is by having the text `#!/usr/bin/bash` in the starting line , this way of telling the bash to execute is a normal way but newline character was new to me, If it was new to you, share to others and enjoy!!!.

Will see ya in the next blog. Thank you.
