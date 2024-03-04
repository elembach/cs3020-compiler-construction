# Midterm Review (Spring 2024)

# Format

- Open notes: bring as much printed material as you want
- Closed computers / phones / everything else
- You'll be expected to be familiar with:
  - Languages (and IRs) used in our compilers
  - Compiler passes in our assignments
- Should take only 30-40 minutes
- If you successfully completed the homework assignments, the questions should be easy
- Liberal partial credit applied (especially when you're asked to write code)

# Major topics

- Atomicity & removing complex operands (Remove-complex-opera*)
- x86 programming
- Translation to x86 (Select-instructions)
- Passes of the A2 / A3 / A4 compiler
- Stack allocation
- Register allocation
  - Liveness analysis
  - Interference graph
  - Allocation by graph coloring
- Typechecking
- Control flow graphs and continuations

# Sample Questions

## Compiler Passes

For each of the following passes, what is its purpose? What property
is true of its output?

- `remove-complex-opera`
- `select-instructions`
- `assign-homes` (A2)
- `allocate-registers` (A3)
- `typecheck` (A4)
- `explicate-control` (A4)
- `patch-instructions`
- `print-x86`

## Removing Complex Operands

Which of the following are atomic expressions?

    1
    True
    5 + 6

Perform the remove-complex-opera* pass on this program:

    x = 5 + 6
    y = x + 7 + 8
    print(x + y)

    x = 5 + 6
    tmp1 = x + 7
    y = tmp1 + 8
    tmp2 = x + y

## x86 / compiler passes / stack allocation

Compile the pseudo-x86 program below into x86, placing all variables
on the stack.

    movq $1, x
    movq $4, tmp_1
    addq $5, tmp_1
    movq tmp_1, tmp_2
    addq x, tmp_2
    movq tmp_2, %rdi
    callq print_int
    


Part (a): for each variable in the program, assign the variable a home
on the stack.

- `x`: -8(%rbp)
- `tmp_1`: -16(%rbp)
- `tmp_2`: -24(%rbp)

Part (b): what is the size (in bytes) of the stack frame for this
program?

align(8*3) = 32 (3 variables)

Part (c): write down the complete x86 program

## Liveness analysis / interference graph / graph coloring

Consider this pseudo-x86 program (same as above). Annotate each
instruction with its *live-after set*.

start at the end and go backwards
looks at instr i+1 to compute live-after for instr 1
use the equation: remove everything written, then add everything read

addq reads both

    movq $1, x          {x}
    movq $4, tmp_1      {x, tmp_1}
    addq $5, tmp_1      {x, tmp_1}
    movq tmp_1, tmp_2   {x, tmp_2}
    addq x, tmp_2       {tmp_2}
    movq tmp_2, %rdi    {}
    callq print_int     {}

Draw the interference graph for this program.

interference graph: edge between writtem vars of instr and all of its live-afters

          x
        /   \
      /       \
  tmp_1       tmp_2

For the following interference graph, assign a register to each
variable using saturation-based graph coloring. You may use **only**
the registers `rbx` and `rcx`.
x: 1{0 }
tmp_1: 0{1 }
tmp_2: 0{1 }


color map:
0: rbx
1: rcx

- `x`:  rcx
- `tmp_1`: rbx
- `tmp_2`: rbx

## Typechecking

For each of the following programs, determine whether or not the
program is *well-typed*. If it is not well-typed, circle the part of
the program containing a type error.


    x = 5
    print(x)
    
    x = 5
    x = 6
    print(x)
    
    x = 5
    x = True
    print(x)

    X HAS TWO TYPES HERE(x = True)^^
    
    x = 5 + True
    print(x)

     X HAS TWO TYPES HERE(True)^^
    
    if 5 == 6:
        x = 5
    else:
        x = True

    TYPE ERROR(there should be a the same type in both branches)^^

## Control Flow and Control Flow Graphs

For the following program, draw the control flow graph in Cif output
by the *explicate-control* pass.

Responsible for un-nesting if statements

    if 3 > 4:
        if 4 < 5:
            x = 1
        else:
            x = 2
    else:
        x = 3

                 3>4

        4<5

 x=2        x=2               x=3
    \       /
      \   / 
        0

                 retq


