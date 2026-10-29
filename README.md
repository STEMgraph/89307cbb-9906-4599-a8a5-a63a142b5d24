<!---
{
  "id": "89307cbb-9906-4599-a8a5-a63a142b5d24",
  "teaches": "Cpp: ostream, filebuf and std::cout",
  "depends_on": [
    "302c98a7-cbea-435c-ada2-bbf7538429a2",
    "657b4539-28e0-4994-a411-fa19fff70309"
  ],
  "author": "Stephan Bökelmann",
  "first_used": "2026-10-29",
  "keywords": [
    "C++",
    "iostream",
    "ostream",
    "filebuf",
    "std::cout",
    "operator<<",
    "buffering",
    "g++"
  ]
}
--->

# Cpp: ostream, filebuf and std::cout

> In this exercise you build `std::cout` from its parts: a buffer on a file, a stream on top of it, and an overloaded `<<`. Then you see that `std::cout` is just a ready-made object of that class, which exists before `main` starts. No `sudo` is required.

## Introduction

In C, `printf` writes to the global `FILE *stdout`. C++ separates two jobs that `FILE` does at once: a **stream buffer** (`std::streambuf`, for files `std::filebuf`) owns the destination and the buffering, an **`std::ostream`** sits on top of a buffer and offers the formatting interface. The operator `<<` is nothing special: it is an ordinary overloaded operator of the class `std::ostream` that returns the stream itself, which is why calls can be chained.

`std::cout`, `std::cerr` and `std::cin` are objects of these classes that the standard library creates for you before `main` runs. Everything you learn about `ostream` therefore applies to `std::cout`, and every function that accepts an `std::ostream&` accepts the console, a file or a string alike.

### 1.1) Further Readings and Other Sources

* [cppreference: std::basic_ostream](https://en.cppreference.com/w/cpp/io/basic_ostream)
* [cppreference: std::basic_filebuf](https://en.cppreference.com/w/cpp/io/basic_filebuf)
* [cppreference: std::cout, std::cerr](https://en.cppreference.com/w/cpp/io/cout)

## Tasks

### Task 1: A terminal is a file

```bash
mkdir -p ~/cpp-stream && cd ~/cpp-stream
```

```c++
// file: hello1.cpp
#include <iostream>
#include <fstream>

int main() {
    std::filebuf fb;
    fb.open("/dev/tty", std::ios::out);
    std::ostream os(&fb);
    os.write("Hello Cpp!", 10);
    os.put('\n');
    fb.close();
}
```

```bash
g++ -std=c++17 -Wall -Wextra hello1.cpp -o hello1 && ./hello1
./hello1 > out.txt; cat out.txt
```

The text appears on the terminal even though you redirected the output. Why? Which file did the program open?

### Task 2: Formatting with `<<`

Replace the two lines after the `ostream` declaration by:

```c++
    os << "Hello Cpp! " << 42 << ' ' << 3.14 << std::endl;
```

Compile and run it. `write` and `put` output raw bytes; `<<` converts values to text. Look at the line with the chained `<<`: each `<<` returns the same stream, so `(os << "a") << 42` works. Write down what the return type of `operator<<` must be.

### Task 3: Use the ready-made object

```c++
// file: hello3.cpp
#include <iostream>

int main() {
    std::cout << "Hello Cpp!" << std::endl;
    std::cout.write("raw bytes\n", 10);
    std::cout.put('!');
    std::cout.put('\n');
    std::cout << "address of cout: " << &std::cout << std::endl;
}
```

Run it. `std::cout` is an object: you can call member functions on it and take its address. It does not need a `filebuf` in your code, the library has already opened standard output for it.

### Task 4: One function, many destinations

```c++
// file: greet.cpp
#include <fstream>
#include <iostream>
#include <sstream>

void greet(std::ostream &os) {
    os << "Hello from greet()" << std::endl;
}

int main() {
    greet(std::cout);

    std::ofstream file("greeting.txt");
    greet(file);

    std::ostringstream text;
    greet(text);
    std::cout << "string holds: " << text.str();
}
```

```bash
g++ -std=c++17 -Wall -Wextra greet.cpp -o greet && ./greet && cat greeting.txt
```

`ofstream` and `ostringstream` are derived classes of `ostream` (you will see inheritance in a later exercise). A function written for the base class works with all of them.

### Task 5: Two streams, one terminal

```c++
// file: streams.cpp
#include <iostream>

int main() {
    std::cout << "to stdout\n";
    std::cerr << "to stderr\n";
}
```

```bash
g++ -std=c++17 -Wall -Wextra streams.cpp -o streams
./streams
./streams > out.txt
./streams 2> err.txt
./streams > /dev/null
```

For each call, note which line still appears on the terminal and what is in the files. Compare with the stream redirection exercise from the shell.

### Task 6: `std::endl` or `'\n'`?

```c++
// file: flush.cpp
#include <cstdlib>
#include <iostream>

int main(int argc, char **) {
    for (int i = 0; i < 3; ++i) {
        if (argc > 1) std::cout << "line " << i << std::endl;
        else          std::cout << "line " << i << '\n';
    }
    std::abort();                       // the program dies here
}
```

```bash
g++ -std=c++17 -Wall -Wextra flush.cpp -o flush
./flush      > a.txt; wc -l a.txt
./flush endl > b.txt; wc -l b.txt
```

`std::endl` writes a newline **and flushes** the buffer, `'\n'` only writes the newline. When the output goes to a file, the text waits in a buffer until it is full or flushed; `abort()` kills the program without emptying it. One file contains three lines, the other none. If `strace` is installed on your machine, you can count the `write` system calls of a longer loop with `strace -e trace=write ./flush endl > /dev/null`. Explain why `std::endl` costs time when you write many lines to a file, and why it helps just before a crash.

### Task 7: Objects that exist before `main`

```c++
// file: before.cpp
#include <iostream>

struct Probe {
    const char *name;
    Probe(const char *n) : name(n) { std::cout << "  + " << name << " constructed\n"; }
    ~Probe()                        { std::cout << "  - " << name << " destroyed\n"; }
};

Probe global("global");

int main() {
    std::cout << "main begins\n";
    Probe local("local");
    std::cout << "main ends\n";
}
```

```bash
g++ -std=c++17 -Wall -Wextra before.cpp -o before && ./before
```

The global object is constructed before `main` and destroyed after it. `std::cout` is created and destroyed in the same way. That is why you may use it from the first line of `main` to the last, and why you must not rely on it inside constructors of other global objects across files without care.

## Questions

- What does a `filebuf` do, and what does an `ostream` add to it?
- Why can `operator<<` be chained? What does it return?
- In Task 1, why does the text still arrive on the terminal after `> out.txt`?
- What is the difference between `std::cout` and `std::cerr` from the point of view of redirection?
- What is the practical difference between `std::endl` and `'\n'`?
- Which object is constructed first in Task 7, `global` or `std::cout`? What happens if `Probe`'s constructor did not find `std::cout` ready?

## Advice

Prefer `'\n'` and use `std::endl` only where you really want the flush, for example before a program might crash or before waiting for input. In your own functions that print something, take an `std::ostream&` instead of writing to `std::cout` directly: the function becomes testable and reusable for files.
