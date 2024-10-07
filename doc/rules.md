# Variables, Functions, Rules and a exampel with vpath
In following we will go through `make` variables, functions and rules. We will also visit an upated
example of the `hello world` program.

## Variables
A variable is a name defined in a makefile to represent a string of text, called the variable's
value. These values are substitued by explicit request into targets, prerequisites, recipes, and
other parts of the makefile. By convention are variables that are internal to the makefile lowercased.
Variables that might be set from the command line are uppercased.

### Variable References
A variable name must be surrounded by $() or ${} to be recognized by make. Variables are case-sensitive,
hence $(CC) and $(cc) refer to two different variables. 
```makefile
objects = program.o foo.o utils.o
program : $(objects)
    cc -o program $(objects)
    
$(objects) : defs.h
```
One type of variable that do not require parentheses is the single character variable.

### Simple Variable
A *simply expanded* variable is defined using ':=' assignment operator. Any make variable references
in the righthand side are expanded and the resulting text saved as the value of the variable upon 
reading the line from the `makefile`. The variable is not updated again however many times it is
referenced.
```makefile
MAKE_DEPEND := $(CC) -M
```

### Recursive Variable
The variable `$(object)` in the example above is a *recursively expanded* variable. Variables of this
sort are defined by using '='. For a *recursive variable* the righthandside is read without being
expanded. The expansion of the variable happens when it is used. The variable is re-evaluated upon
every time it is used. Consequently, the content of a *recursive variable* may change during
the course of a `makefile`.

### Conditional Variable
```makefile
# Put all generated files in the directory $(PROJECT_DIR)/bin.
OUTPUT_DIR ?= $(PROJECT_DIR)/bin
```
The '?=' operator is called the *conditional variable assignment operator*. Here the the assignment is
performed only if the variable does not yet have a value.

### Append variable
The append assignment operator is '+='. This operator appends text to a variable. This operator is
particularly useful for collecting values into a variable incrementally.
```makefile
CPPFLAGS += -DUSE_NEW_MALLOC=1
```

### Target variable
You can define a variable for a specific target, such that the variable is valid only for the target:
```makefile
gui.o: CPPFLAGS += -DUSE_NEW_MALLOC=1
gui.o: gui.h
```
While the *gui.o* target is being processed, the valu of CPPFLAGS will contain -DUSE_NEW_MALLOC=1 in
addition to its original contents. When the *gui.o* target is finished, CPPFLAGS will be set back to
its original value. The general syntax for target-specific variables is:
```makefile
target...: variable = value
target...: variable := value
target...: variable += value
target...: variable ?= value
```

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


## Macro
```makefile
BIN    := /usr/bin
PRINTF := $(BIN)/printf
DF     := $(BIN)/df
AWK    :- $(BIN)/awk

define free-space
    $(PRINTF) "Free disk space "
    $(DF) . | $(AWK) 'NR == 2 { print $$4 }'
endef
```
With `define - endef` command we can define a multiline variable, a variable that contains a script. 
In the above example we have two-line script. With `define` the variable can contain embedded newlines.
Conseqeuently, we call it a macro to disguish from a single line variable.


## Rules
The rule governing the building of the hello world program was so called _explicit_ rule. There are other
types of rules as well, and we will be going through these now:
- explicit rules
- pattern rules
- implicit rules

### Explict rules

#### wildcards
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

#### empty targets
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

### Pattern rules
```makefile
%.o : %.c
	$(CC) $(CFLAGS) -c $< -o $@
```
A pattern rule looks like an ordinary rule, except that its target contains the character '%'. The target
is considered a pattern for matching file names; the '%' can match any nonempty substring, while other
characters match only themselves. The shown rule '%.o : %.c' says how to make any file `stem.o` from another
file `stem.c`

### Implicit rules
`make` has about 90 built-in implicit rules.  There are built-in pattern rules for C, C++, Pascal, Fortran,
Modula, Texinfo, TEX, Emacs, Lisp, RCS and SCCS. The two last ones are version control systems. The implicit
rules database can be listed with the command:
```sh
make --print-data-base # This gives a long output!
```
The implicit rule for building an executable from object files looks like this:
```makefile
%: %.o
#  commands to execute (built-in):
	$(LINK.o) $^ $(LOADLIBES) $(LDLIBS) -o $@
```
A interesting note is that this rule is not part of the implicti rulebase on my Mac with
make version 3.81, but it exists in the implicit rulebase on Linux with make version 4.3


### An updated "Hello World!" example - introducing vpath
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

- macro
- functions
