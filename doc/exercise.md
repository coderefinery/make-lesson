# Exercise

## Objective
- To try out the `make` command on different makefiles.

## Steps
In this exercise we will execute `make` on different makefiles.
You will also edit an Makefile and try to create your own Makefile.

**Step 1**: Logon on to a Linux system like Dardel
**Step 2**: Go to [makefile-examples](https://github.com/coderefinery/makefile-examples) repository
and clone the repository to your user space on the Linux system.
**Step 3**: Change to the subdirectory `make-examples`, list the files. You will see that there is
4 makefile examples in the repository:
```sh
git clone https://github.com/coderefinery/makefile-examples.git
cd makefile-examples
ls
```
**Step 4**: Change to subdirectory `example_1` and execute make. You will see that `make` executes
the command to build the executable. Test the executable and then remove it.
```sh
cd example_1
make
./hello.exe
rm hello.exe
```
**Step 5**: Introduce an error in the `example_1` makefile. Open the `Makefile` in an editor and
replace the tab in front of the command for the target with spaces. Save the file and execute
`make` again. This time you get an error:Makefile:2: *** missing separator.  Stop.
