# Rules
The rule governing the building of the hello world program was so called _explicit_ rule. There are other
types of rules as well, and we will be going through these now:
- explicit rules
- pattern rules
- static pattern rules
- suffix rules
- implicit rules

## Explict rules

### wildcards
`make` supports wildcards, also known as globbing.`make`'s wildcards are identical to the Bourne shell's.
Instead of listing all the files in a program explicitly, you can use wildcards together with automatic
variables (We will going through automatic variables in a moment):

```makefile
prog: *.c
    $(CC) -o $@ $^
```

### phony targets
A _phony target_ are targets that do not represent files. Here is a phony target `clean`:
```makefile
.PHONY: clean

clean:
    rm -f *.o
```

This tells `make` that a target is not real file. Phony targets are always out of date. Consequently, the
command will always be executed.

Many _makefiles_ include a set of standard phony targes. The table list these:

| target      | function                                                                  |
| ------      | --------                                                                  |
| `all`       | Perform all tasks to build the application                                |
| `install`   | Create an installation of the application from the complied binaries      |
| `clean`     | Delete the binary files generated from source                             |
| `distclean` | Delete all the generated files that were not in the original distribution |
| `TAGS`      | Create a tags table for use by editors                                    |
| `info`      | Create GNU info files from their Texinfo sources                          |
| `check`     | Run any tests associated with this application                            |
|             |                                                                           |

### empty targets
An _empty target_ is similar to phony targets; it is used to hold recipes for an action that you request
explicitly. The purpose of the empty target file is to record when the rule's recipe was last executed.
It does so because one of the commands in the recipe is a `touch` command to update the target file.

Here is an example:
```makefile
print: foo.c bar.c
    lpr -p $?
    touch print
```

The command `make print`  will execute the `lpr` command if one of the source file has changed since the
last `make print`. The automatic variable `$?` is used to print only those files that have changed.
(We will discuss automatic variables in a moment)


## Variables
A variable have the syntax:
```makefile
$(variable-name)
```
A variable name must be surrounded by $() or ${} to be recognized by make. Variables are case-sensitive,
hence $(CC) and $(cc) refer to two different variables. One type of variable that do
not require parentheses is the single character variable. By convention are variables that are internal
to the makefile lowercased. Variables that might be set from the command line are uppercased.

### Automatic variables
There are seven automatic variables. Automatic variables are set by `make` after a rules is matched.
They provide access to elements from the target and prerequisite lists. It this way you don't have
to explictily specify any filenames.

| Variable | Function                                                                           |
| -------- | --------                                                                           |
| $@       | The filename representing the target                                               |
| $%       | The filename element of an archive member specification                            |
| $<       | The filename of the first prerequisite                                             |
| $?       | The names of all prerequisites that are newer than the target, separated by spaces |
| $^       | The filenames all the prerequisites, separated by spaces                           |
| $+       | Similar to $^ except that $+ include duplicates                                    |
| $*       | The stem of the target filename.                                                   |

These seven variables have variants that get just the file's directory name or just the file name within
the directory. The variant variables names are formed by appending 'D' or 'F'. These variant variables are 
more than one character long and so must be enclosed in parentheses, i.e. $(@D), $(@F). 
See [GNU Make Automatic variables](https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html)

### vpath
In the following example we have reworked "Hello world"-example. The source files reside in a subdirectory `src`.
The printf statement is moved to a function hello(), which is defined in a new file `module.c`.

    directory/
     |-makefile
     |-include/
     |
     |-src/-
     |   |
     |   |-hello.c
     |   |
     |   |-module.c
     |
     |-doc/
     
module.c:     
```c
#include<stdio.h>

void hello()
{
  printf("Hello, world!\n");
}

```

hello.c:
```c
#include<stdlib.h>

void hello();

int main(int argc, char *argv[])
{
  hello();
  return EXIT_SUCCESS;
}

```

The new makefile includes phony targets and pattern rules. It it also depend upon so called implicit rules.
```makefile
.PHONY: clean all install

vpath %.c src

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

hello: hello.o module.o

clean:
	rm *.o hello

install: hello
	mkdir -p ./bin/
	cp hello ./bin/

all: hello
```

The phony targets are remade every time they are called like `make clean`. We have a new directive `vpath`.
This tells `make` to look for C-files in the src subdirectory. When `make hello` is called, `make` searches
the current directory for C-files, but none is found since they are all in the `src` subdirectory. With
the `vpath`-directive, make goes on to search for C-files in `src`.

Next we have a pattern rule. The `%` character match any sequence of zero or more characters. Here it
matches the stem of the source filenames.

Following the pattern rule we have a rule without a command line. When `make` look for how to make `hello`,
it will go to the implicit rules database. There is approximately 90 different implicit rules in make.
One of them is how to make a executable from binary object files:
```makefile
%: %.o
#  commands to execute (built-in):
	$(LINK.o) $^ $(LOADLIBES) $(LDLIBS) -o $@
```

This tells `make` to invoke the linker to link all the prerequisites together with possible other libraries
to produce the target executable.

- simple expanded variable
- recursively expanded variable
- conditional variable
- append variable
- target variable
