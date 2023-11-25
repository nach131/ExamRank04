# Microshell

<details>
  <summary>subject</summary>

```txt
Assignment name  : microshell
Expected files   : .c .h
Allowed functions: malloc, free, write, close, fork, waitpid, signal, kill, exit, chdir, execve, dup, dup2, pipe, strcmp, strncmp
--------------------------------------------------------------------------------------

Write a program that will behave like executing a shell command
- The command line to execute will be the arguments of this program
- Executable's path will be absolute or relative but your program must not build a path (from the PATH variable for example)
- You must implement "|" and ";" like in bash
	- we will never try a "|" immediately followed or preceded by nothing or "|" or ";"
- Your program must implement the built-in command cd only with a path as argument (no '-' or without parameters)
	- if cd has the wrong number of argument your program should print in STDERR "error: cd: bad arguments" followed by a '\n'
	- if cd failed your program should print in STDERR "error: cd: cannot change directory to path_to_change" followed by a '\n' with path_to_change replaced by the argument to cd
	- a cd command will never be immediately followed or preceded by a "|"
- You don't need to manage any type of wildcards (*, ~ etc...)
- You don't need to manage environment variables ($BLA ...)
- If a system call, except execve and chdir, returns an error your program should immediatly print "error: fatal" in STDERR followed by a '\n' and the program should exit
- If execve failed you should print "error: cannot execute executable_that_failed" in STDERR followed by a '\n' with executable_that_failed replaced with the path of the failed executable (It should be the first argument of execve)
- Your program should be able to manage more than hundreds of "|" even if we limit the number of "open files" to less than 30.

for example this should work:
$>./microshell /bin/ls "|" /usr/bin/grep microshell ";" /bin/echo i love my microshell
microshell
i love my microshell
$>

Hints:
Don't forget to pass the environment variable to execve
```
</details>


## microshell.c

```c
#include "microshell.h"

int err(char *str)
{
	while (*str)
		write(2, str++, 1);
	return 1;
}

int cd(char **cmd, int i)
{
	if (i != 2)
		return err("error: cd: bad arguments\n");
	else if (chdir(cmd[1]) == -1)
		return err("error: cd: cannot change directory to "), err(cmd[1]), err("\n");
	return 0;
}

int exec(char **cmd, char **env, int i)
{
	int fd[2];
	int status;
	int has_pipe = cmd[i] && !strcmp(cmd[i], "|");

	if (has_pipe && pipe(fd) == -1)
		return err("error: fatal\n");

	int pid = fork();
	if (!pid)
	{
		cmd[i] = 0;
		if (has_pipe && (dup2(fd[1], 1) == -1 ||
						 close(fd[0]) == -1 ||
						 close(fd[1]) == -1))
			return err("error: fatal\n");
		execve(*cmd, cmd, env);
		return err("error: cannot execute "), err(*cmd), err("\n");
	}

	waitpid(pid, &status, 0);
	if (has_pipe && (dup2(fd[0], 0) == -1 ||
					 close(fd[0]) == -1 ||
					 close(fd[1]) == -1))
		return err("error: fatal\n");
	return WIFEXITED(status) && WEXITSTATUS(status);
}

int main(int n, char **cmd, char **env)
{
	int i = 0;
	int status = 0;

	if (n > 1)
	{
		while (cmd[i] && cmd[++i])
		{
			cmd += i;
			i = 0;
			while (cmd[i] && strcmp(cmd[i], "|") && strcmp(cmd[i], ";"))
				i++;
			if (!strcmp(*cmd, "cd"))
				status = cd(cmd, i);
			else if (i)
				status = exec(cmd, env, i);
		}
	}
	return status;
}

```

## microshell.h

```c

#ifndef MICROSHELL_H
#define MICROSHELL_H

#include <string.h>
#include <unistd.h>
#include <sys/wait.h>

#endif

```