# Exercise

## Objective
- To try out the `make` command on different makefiles.

## Steps
In this exercise we will execute `make` on different makefiles.
You will also edit an Makefile and try to create your own Makefile.

### Example 1
- **Step 1**: Logon on to a Linux system like Dardel
- **Step 2**: Go to [makefile-examples](https://github.com/coderefinery/makefile-examples) repository
and clone the repository to your user space on the Linux system.
- **Step 3**: Change to the subdirectory `make-examples`, list the files. You will see that there is
4 makefile examples in the repository:
```sh
git clone https://github.com/coderefinery/makefile-examples.git
cd makefile-examples
ls
```
- **Step 4**: Change to subdirectory `example_1`. Take a look at the `makefile` and execute make. You will see that `make` executes
the command to build the executable. Test the executable and then remove it.
```sh
cd example_1
cat Makefile
make
./hello.exe
rm hello.exe
```
- **Step 5**: Introduce an error in the `example_1` makefile. Open the `Makefile` in an editor and
replace the tab in front of the command for the target with spaces. Save the file and execute
`make` again. This time you get an error:`Makefile:2: *** missing separator.  Stop.`

### Example 2
- **Step 1**: Change directory to `example_2`. Take a look at the `Makefile` and execute `make`.
See how different object files are created in the subdirectory.
```sh
# from subdirectory example_1
cd ../example_2
ls
cat Makefile
make
ls
```

- **Step 2**: Remove a object file and rerun `make`. Observe how `make` only build the missing
object file and the rebuilds the executable since it is dependent on the newly built object file.
```sh
rm module.o
make
```

- **Step 3**: Remove the executable and rerun `make`. Observe that only the step to build the
executable is taken. The object files that the executable is dependent on is untouched.
```sh
rm hello.exe
make
```

### Example 3
Here you will try to create a makefile. Change into the subdirectory and observe that you have
source files in the subdirectory `src`. How will you start out? The `Makefile` in example 2
build an executable from source files in a `src` subdirectory. Let us see if we can use it as a
starting point.

- **Step 1**: Copy the `Makefile` from example 2 and execute make. You see that you get an error message
from make: `make: *** No rule to make target `hello.o', needed by `hello.exe'.  Stop.`
- **Step 2**: Open an editor (nano, vim) and replace references to `hello.exe` in the `Makefile`
with `calculation.exe`, both as a target as a dependency. Execute `make` again and observe the
erro message: `src/calculation.c:4:10: fatal error: 'example_math.h' file not found`
This is an error message from the compilation of `calculation.c`. The compiler cannot find
the include file `example_math.h` which resides in the subdirectory `include`.
- **Step 3**: To find the include file, the compile needs to be told to look in the `include` subdirectory.
We do this by adding the CFLAGS=-I include to the `Makefile`. Add it at the top of the `Makefile`, after
the .PHONY statement, like this:
```makefile
.PHONY: clean all install
CFLAGS=-I include
```

- **Step 4**: Rerun `make` and observe how the build of `calculation.c` completes, but the build
of the executable fails due to missing of `module.o`:
`make: *** No rule to make target module.o, needed by calculation.exe.  Stop.`
```sh
make
```

- **Step 5**: Edit the `makefile` again and replace `module.o` with `sin.o cos.o` as
dependencies for `calculatione.exe`.
```makefile
# The calculation.exe target in the Makefile
calculation.exe: calculation.o sin.o cos.o
	$(CC) $(CFLAGS) $^ -o $@
```
```sh
make
ls
./calculation.exe
make clean
make
```

### Example 4
In this example the builds, the object files and the executable, ends up in its own subdirectory `bin`.
This separates the source files and the top `Makefile` from the binaries. This is very tidy and useful.
Take a look at the make file, see how a function creates the necessary subdirectory, and how binaries
are placed in an own subdirectory:
```sh
cd ../example_4
ls
cat Makefile
make
touch src/cos.c  # simulate a update of src/cos.c
make
make clean
ls
```
