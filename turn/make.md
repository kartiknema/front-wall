
# Make

Make is one of the oldest build systems. It uses Makefiles to determine what needs to be built and how. In the case of a large program, recompiling it entirely every time some small change is made will be inefficient. Make uses Makefiles to determine which parts of the large program need to be recompiled, i.e. it does not recompile the entire program.

When should a file be recompiled?
Suppose a file, file1.cpp depends on the files file2.cpp and file3.cpp. If changes are made to file3.cpp, then file.cpp will also need to be recompiled, in general a file needs to be recompiled if it changes or any of its dependencies changes.

Alternatives to Make: Bazel, CMake, Ninja.

## Makefile Syntax
A Makefile contains a collection (or bundle) of rules, where each rule has 3 attributes:
- Target: The file to be generated (example an executable).
- Dependencies or Prerequisites:  Files that the target depends on (example: source files). The files declared as dependencies need to exist, else the Commands for the target won’t run.
- Commands: Shell commands (or more generally, the series of steps) to build the target.

## Structure of a Makefile rule:
```text
target: dependencies
	command (notice the tab indentation)
	command
	command
```

Make uses filesystem timestamps (last modified at), to determine if something has changed.

Say we have a rule:

```text
blah: blah.c
	gcc blah.c -o blah
```

Here the target blah depends on the file named “blah.c”, if the file “blah.c” is changed then the target “blah” should also be recompiled, i.e. the commands for the target should be run again. 

## How does Make determine if it should run a target?
If the target file does not exist, then Make will always run the rule, if the target file exists then Make checks if any of the dependencies of the target have changed, i.e. if any of the target dependencies are newer than the target (by using timestamps). If such a dependency is found, then the target will be run again (i.e. the commands for the target will run again). If the target specifies no dependencies, then it will only be run if the target file does not exist.

## Efficient Recompiling:
How Makefile rules are written dictate how quickly the program can be recompiled. As stated before only those parts of the program which have changed should be recompiled, rather than recompiling everything, this is governed by the Makefiles.

Consider 2 ways of writing a Makefile to compile a program:

Approach 1:
```text
myapp: main.o utils.o
    g++ main.o utils.o -o myapp
main.o: main.cpp
    g++ -c main.cpp
utils.o: utils.cpp
    g++ -c utils.cpp
```

Approach 2:
```text
myapp: main.cpp utils.cpp
    g++ main.cpp utils.cpp -o myapp
```

Approach 2 looks the simpler one, however it is also in-efficient, in command used in approach 2, internally the following things take place:
- The files main.cpp and utils.cpp are compiled into object code, i.e. (.o) files
- Once the sources are compiled, the resulting object files are linked into the executable

If any of the dependencies of the target: myapp (in this case main.cpp and utils.cpp) changes then the target needs to be rebuilt, however the command here (g++ main.cpp utils.cpp -o myapp) recompiles both the source files, all the time. This is not desirable, especially for large projects.

A better approach is the first one, where we make use of object files for building the executable, and specify separate rules for building the object files themselves. In this approach, say if utils.cpp is changed, then utils.o will still need to be  rebuilt, however main.o remains untouched (since main.c did not change). Hence instead of compiling both source files, we just compile the one which has changed, while continuing to use the already existing object files for the other sources.

Refer: https://makefiletutorial.com/
