# IDSDFK
A Turing Tarpit of an esoteric programming language vaguely inspired by SleepSort.

The name "IDSDFK" is an agglomeration of the initial letters of the types of instructions this language has, of which there are six. They are:
1. Increment (`+`)
2. Decrement (`-`)
3. Sleep (`_`)
4. Define (`:`)
5. Fork (`#`)
6. Kill (`/`)

(A seventh instruction type, print (`!`), is available with the `-d` debugging flag in the interpreter, although this contravenes the strict language specification.)

Before I get into what each of these mean, it's more important to understand the memory and execution models the language operates on. There are two types of units in memory, *variables* and *subroutines*. Variables hold unbounded integer values, and are automatically initialized to zero. Subroutines hold arbitrary code, which can be run via the fork instruction (`#`), and are automatically initialized to do nothing when forked. Both variables and subroutines can further be subdivided into two categories: *registers*, which are named by string values (not including the characters `+-_:#/()` or whitespace and not beginning with `@`), and *locations*, which are named by integers. Variables and subroutines are stored in separate namespaces, so their names cannot conflict (although generally variable registers will use lowercase and subroutine registers will begin with capital letters, per convention). There are two special variable registers, `<` and `>`, used for input and output respectively, and two corresponding special subroutine registers with the same names.

The execution model consists of a set of *timesteps*, each of which holds a queue of *actions*, typically (but not always) instructions. For each timestep in numerical order, the actions in the queue are retrieved and executed in order. Each action corresponding to an instruction, after doing what it is intended to do, appends an action corresponding to the following instruction in the code to an action queue, typically the one for the current timestep (unless the instruction is sleep (`_`)). Certain instructions can modify the action queues in other ways, including by appending extra actions and by removing specific actions. After there are no more actions in a specific timestep's queue, execution advances to the next timestep with a non-empty cue. Execution can never go backward, and action queues from previous timesteps are inaccessible (or simply deleted, depending on the implementation).

Each instruction consists of two components, a symbol (one of `+-_:#/`) and any number of arguments (the specific number depending on the symbol). Depending on the instruction, the arguments can consist of *names* and/or *code blocks*. Code blocks are delimited by parentheses, whereas names consist of essentially everything except symbols and delineated code blocks. Name arguments, which correspond to register names, have the same syntactical requirements, with one exception: IDSDFK ignores whitespace completely, so a name argument may be specified with intervening whitespace (which will be stripped). For example, in the instruction `:QWERTYU  IOP(/0)`, `:` is the instruction symbol (a define instruction), `QWERTYU  IOP` is a name argument corresponding to a register named `QWERTYUIOP`, and `(/0)` is a code block argument. (Note that there is currently no way to specify two consecutive names as arguments to a command; this feature is not used by any instruction, but could be necessary in future language extensions.) Prefacing a name argument `n` with `@` indicates that the instruction should operate on the variable or subroutine location specified by the value of the variable register `n`, rather than on the variable or subroutine register `n` itself.

And now, without further ado, the types of instructions:

1. Increment (`+`)

 - Takes one name argument and increments the corresponding variable.

2. Decrement (`-`)

 - Takes one name argument and decrements the corresponding variable.

3. Sleep (`_`)

 - Takes one name argument and delays execution by appending the following instruction to the queue `n` timesteps in the future (instead of the current timestep's queue), where `n` is the value of the variable referenced by the instruction's argument. Attempting to sleep for zero timesteps is a useful no-op. Negative values are also treated as zero.

4. Define (`:`)

 - The only dyadic instruction type. Takes a name argument and a code block argument in that order, and assigns the code block to the subroutine specified by the name argument.

5. Fork (`#`)

 - Takes a name argument and executes the corresponding subroutine, appending the first instruction of the subroutine to the current timestep's action queue prior to appending the next instruction in the parent execution path. There is no way to specify "arguments" to a subroutine, unlike in many languages.

6. Kill (`/`)

 - Takes a name argument and kills all running instances of the corresponding subroutine by removing all corresponding actions in the present and all future action queues. Note that killing a "parent" subroutine has no effect on "child" subroutines. Killing a subroutine that is not running has no effect.

7. Print (`!`)

 - Only available in debugging mode (`-d` flag). Takes a name argument and prints the name and value of the corresponding variable. There is currently no such option for subroutines.

Input and output have a few special characteristics. The instruction to get a character of input is `#<`, which forks the special subroutine `<` which writes a character of input to the variable register `<`. Similarly, the instruction to write a character of output, `#>`, forks the special subroutine `>` to write a character of output from the variable register `>`. It is important to note that in terms of the execution model, `#<` and `#>` behave like any other fork instruction in that they append the new subroutine's "first instruction", in these cases an action not corresponding to an instruction, to the current action queue, which is then executed only once it reaches the front of the queue. This execution is not immediate, and there is time for the output register to be modified by other instructions in the interim (or alternately for modifications to the input register in the interim to be lost). Similarly, before they are executed, the `<` and `>` subroutines can be "killed" like any other subroutine. Input and output are both done in terms of bytes, and the value of the output register is taken modulo 256 (with a positive result) before being output. Empty input/EOF are represented by a zero byte. In all other respects, `<` and `>` may be read and written like any other registers.

The main subroutine is specified by the subroutine register `0`. It may be recursively forked or killed like any other subroutine (again, `/0` will not always work as a kill switch for the program, as it does not immediately terminate child subroutines). Defining `0` using `:` is undefined behavior and might cause problems with other interpreter features in future versions of the interpreter that actually have features, but will typically work in practice.

As IDSDFK is very permissive in syntax, typing, and memory model, there are very few errors that may occur. The only errors that typically will occur (barring interpreter bugs) are syntax errors from mismatched parentheses, which are reported straightforwardly, or errors from incorrect arguments to commands, which are given as cryptic Python errors in the interpreter given.

Run programs with the command `python interpreter.py example.idk`, where `example.idk` is replaced with the filename of the program you're running. Adding the `-d` flag will enable print instructions (`!`) while disabling register names containing `!`. (Beware of forgetting to remove print instructions once you're done debugging, since they will be interpreted as parts of register names and will almost always break things.) Look in the files packaged in this repository for examples of IDSDFK programs.

How to convince this language to actually do anything is left as an exercise for the reader.
