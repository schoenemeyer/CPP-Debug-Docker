# CPP Debug Docker

## Introduction

These are examples of how to debug common C/C++ bugs, such as logic errors, segmentation faults, and memory leaks, using CMake, [GDB](https://www.gnu.org/software/gdb/) and [Valgrind](https://valgrind.org/) in Docker container. A more comprehensive [tutorial](https://leimao.github.io/blog/Debug-CPP-In-Docker-Container/) could be found on my [website](https://leimao.github.io/).


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
mkdir build
cd build
cmake ..
make
```

### Example Files

* [Logical Error](logicalError.cpp)
* [Core Dumped](coreDumped.cpp)
* [Memory Leak](memoryLeak.cpp)

## References

* [Debug C++ Programs in Docker Container](https://leimao.github.io/blog/Debug-CPP-In-Docker-Container/)
* [How to Debug Using GDB](https://cs.baylor.edu/~donahoo/tools/gdb/tutorial.html)

