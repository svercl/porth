# Porth

**WARNING! This language is a work in progress!**

It's like [Forth](https://en.wikipedia.org/wiki/Forth_(programming_language)) but written in [Python](https://www.python.org/). But I don't actually know since I never programmed in Forth, I only heard that it's some sort of stack-based programming language. Porth is also stack-based programming language. Which makes it just like Forth am I rite?

Porth is planned to be
- [x] Compiled
- [x] Native
- [x] Stack-based (just like Forth)
- [ ] Turing-complete (yes, the development is at such an early stage that this thing is not even Turing complete yet)
- [ ] Statically typed (the type checking is probably gonna be similar to the [WASM validation](https://binji.github.io/posts/webassembly-type-checking/))
- [ ] Self-hosted (Python is used only as an initial bootstrap, once the language is mature enough we gonna rewrite it in itself)

## Example

Simple program that prints numbers from 10 to 1 in a descending order:

```forth
10 while dup 0 > do
  dup .
  1 -
end
```

## Quick Start

### Simulation

Simulation simply interprets the program.

```console
$ cat program.porth
34 35 + .
$ ./porth.py sim program.porth
69
```

### Compilation

Compilation generates assembly code, compiles it with [nasm](https://www.nasm.us/), and then links it with [GNU ld](https://www.gnu.org/software/binutils/). So make sure you have both available in your `$PATH`.

```console
$ cat program.porth
34 35 + .
$ ./porth.py com program.porth
[INFO] Generating ./program.asm
[CMD] nasm -felf64 ./program.asm
[CMD] ld -o ./program ./program.o
$ ./program
69
```

## Language Reference

This is what the language supports so far. **Since the language is a work in progress the exact set of operations is the subject to change.**

### Stack Manipulation

- `<integer>` - push the integer onto the stack. Right now the integer is anything that is parsable by [int](https://docs.python.org/3/library/functions.html#int) function.
```
push(<integer>)
```
- `dup` - duplicate an element on top of the stack.
```
a = pop()
push(a)
push(a)
```
- `.` - print the element on top of the stack to stdout and remove it from the stack.
```
a = pop()
print(a)
```

### Comparison

- `=` - checks if two elements on top of the stack are equal. Removes the elements from the stack and pushes `1` if they are equal and `0` if they are not.
```
a = pop()
b = pop()
push(int(a == b))
```
- `>` - checks if the element below the top greater than the top.
```
b = pop()
a = pop()
push(int(a > b))
```

### Arithmetics

- `+` - sums up two elements on the top of the stack.
```
a = pop()
b = pop()
push(a + b)
```
- `-` - subtracts the top of the stack from the element below.
```
a = pop()
b = pop()
push(b - a)
```

### Control Flow

- `if <then-branch> else <else-branch> end` - pops the element on top of the stack and if the element is not `0` executes the `<then-branch>`, otherwise `<else-branch>`.
- `while <condition> do <body> end` - keeps executing both `<condition>` and `<body>` until `<condition>` produces `0` at the top of the stack. Checking the result of the `<codition>` removes it from the stack.

### Memory

- `mem` - pushes the address of the beginning of the memory where you can read and write onto the stack.
```
push(mem_addr)
```
- `.` - store a given byte at the given address.
```
byte = pop()
addr = pop()
store(addr, byte)
```
- `,` - load a byte from the given address.
```
addr = pop()
byte = load(addr)
push(byte)
```

### System

- `syscall1` - perform a syscall with 1 argument.
```
syscall_number = pop()
<move syscall_number to the corresponding register>
for i in range(1):
    arg = pop()
    <move arg to i-th register according to the call convention>
<perform the syscall>
```
- `syscall3` - perform a syscall with 3 arguments.
```
syscall_number = pop()
<move syscall_number to the corresponding register>
for i in range(3):
    arg = pop()
    <move arg to i-th register according to the call convention>
<perform the syscall>
```
