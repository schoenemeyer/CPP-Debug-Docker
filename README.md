# CPP Debug Docker

## Introduction

These are examples of how to debug common C/C++ bugs, such as logic errors, segmentation faults, and memory leaks, using CMake, [GDB](https://www.gnu.org/software/gdb/) and [Valgrind](https://valgrind.org/) in Docker container. A more comprehensive [tutorial](https://leimao.github.io/blog/Debug-CPP-In-Docker-Container/) could be found on my colleagues  [website](https://leimao.github.io/).


## Docker

### Create Docker Image

```bash
docker build -f cpp.Dockerfile -t cpp-debug:0.0.1 .
```

### Start Docker Container

```bash
docker run -it --rm --cap-add=SYS_PTRACE --security-opt seccomp=unconfined -v $(pwd):/mnt cpp-debug:0.0.1
```

## Examples

### Build Examples

```bash
cd /mnt
mkdir build
cd build
cmake ..
make
```

### Example Files

* [Logical Error](logicalError.cpp)
* [Core Dumped](coreDumped.cpp)
* [Memory Leak](memoryLeak.cpp)

## Run the Logical Error Example

```bash
./logicalError 
This program is used to compute the value of the following series : 
(x^0)/0! + (x^1)/1! + (x^2)/2! + (x^3)/3! + (x^4)/4! + ........ + (x^n)/n! 
Please enter the value of x : 2

Please enter an integer value for n : 3

The value of the series for the values entered is inf

gdb ./logicalError 
(gdb) break ComputeSeriesValue(double, int) 
Breakpoint 1 at 0xc40: file /mnt/header.cpp, line 18.
run
Starting program: /mnt/build/logicalError
This program is used to compute the value of the following series :
(x^0)/0! + (x^1)/1! + (x^2)/2! + (x^3)/3! + (x^4)/4! + ........ + (x^n)/n!
Please enter the value of x : 2

Please enter an integer value for n : 3


Breakpoint 1, ComputeSeriesValue (x=2, n=3) at /mnt/header.cpp:18
18          double seriesValue = 0.0;
(gdb)
```
We suspected that the ComputeFactorial is problematic. We ran the program step by step using next or n several times

```bash
(gdb) next
19          double xpow = 1;
(gdb) n
21          for (int k = 0; k <= n; k ++) {
(gdb) n
22              seriesValue += xpow / ComputeFactorial(k);
(gdb) n
23              xpow = xpow * x;
(gdb) n
21          for (int k = 0; k <= n; k ++) {
(gdb) n
22              seriesValue += xpow / ComputeFactorial(k);
(gdb)
```
When we reach a line of code containing ComputeFactorial, we step into the function ComputeFactorial using step or s.
```bash
(gdb) step
ComputeFactorial (number=1) at /mnt/header.cpp:6
6           int fact = 0;
(gdb)
```
We executed ComputeFactorial further but found the value of fact remains to be 0 before it is returned.
To find the root cause of the problem, we should use backtrace

We could also know where the execution is by viewing the stack using backtrace or bt.

```bash
(gdb) backtrace
#0  ComputeFactorial (number=2) at /mnt/header.cpp:14
#1  0x0000555555554c6f in ComputeSeriesValue (x=2, n=3) at /mnt/header.cpp:22
#2  0x0000555555554b3c in main () at /mnt/logicalError.cpp:25
```

It immediately points to the source of the problem. In header.cpp we set fact intentionally to 0.

```bash
  1 #include "header.h"
  2
  3 int ComputeFactorial(int number) {
  4     // Intentionally set fact = 0 so that the factorial computation is always
  5     // wrong.
  6     int fact = 0;
  7     // The correct setting should be:
  8     // int fact = 1;
  9
 10     for (int j = 1; j <= number; j ++) {
 11         fact = fact * j;
 12     }
 13
 14     return fact;
 15 }

```

## References

* [Debug C++ Programs in Docker Container](https://leimao.github.io/blog/Debug-CPP-In-Docker-Container/)
* [How to Debug Using GDB](https://cs.baylor.edu/~donahoo/tools/gdb/tutorial.html)

