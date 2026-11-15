# Bengali C Compiler (`bnc`)

**Bengali C** is the C programming language written with Bengali keywords. `bnc` is a compiler for it, built with **C and Flex**. It reads a Bengali C program (a `.bn` file), checks it for mistakes, translates it to C, compiles that with GCC and runs it.

```
পূর্ণসংখ্যা প্রধান()                      int main()
{                                       {
    পূর্ণসংখ্যা বয়স = ২৫;                     int v_1 = 25;   /* v_1 = বয়স */
    দেখাও("বয়স: %d\n", বয়স + ১);    ──►      printf("বয়স: %d\n", v_1 + 1);
    ফেরত ০;                                 return 0;
}                                       }
```

Every Bengali keyword maps to exactly one C keyword or function, and programs follow C's structure and rules. The project was built as a compiler design course project and implements every classic compiler phase by hand.

## Features

- **Five visible compiler phases:** lexical analysis, syntax analysis, semantic analysis, code generation and GCC compilation
- **Source-to-source design:** Bengali C is translated to C, and GCC produces the executable
- **Hand-written recursive-descent parser** (no Bison), with panic-mode error recovery so one run reports many errors
- **UTF-8 aware lexer** written in Flex: Bengali letters and digits are matched by their bytes, and Bengali digits (`২৫`) are converted to English digits (`25`)
- **Semantic checks:** declarations and scopes, C type rules, `printf`/`scanf` format checking, array bounds warnings, `থামো`/`পরবর্তী` only inside loops
- **Symbol table** with scopes, and **three-address code** (intermediate code) output
- **Clear error messages** with line numbers and the phase that found the problem
- **Inspection flags** to print tokens, the syntax tree, the symbol table and the intermediate code
- A suite of 17 test programs: correct programs, programs with warnings, and programs with deliberate errors

## How it works

```
 project.bn  (Bengali C)
     │
     ▼
 [1] LEXICAL ANALYSIS      project.l          characters  ->  tokens
 [2] SYNTAX ANALYSIS       parser.c           tokens      ->  syntax tree (AST)
 [3] SEMANTIC ANALYSIS     semantic.c,        types, formats, symbol table
                           symbol_table.c
     INTERMEDIATE CODE     tac.c              three-address code (--tac)
 [4] CODE GENERATION       code_generator.c   syntax tree ->  output.c
 [5] GCC                   gcc -std=c99 output.c
     │
     ▼
 program runs
```

If any phase reports an error, no C code is generated. Invalid source is never silently ignored.

## Project structure

| File | Purpose |
|------|---------|
| `tokens.h` | Token types shared by lexer and parser |
| `project.l` | Flex rules: source text to tokens |
| `ast.h`, `ast.c` | Syntax tree nodes and tree printing |
| `parser.h`, `parser.c` | Recursive-descent parser |
| `symbol_table.h`, `symbol_table.c` | Scoped table of variables |
| `semantic.h`, `semantic.c` | Declarations, type rules, format checking |
| `tac.h`, `tac.c` | Three-address (intermediate) code |
| `code_generator.h`, `code_generator.c` | Syntax tree to C code |
| `main.c` | Driver: runs the phases, calls GCC, runs the program |
| `build.bat`, `run_tests.bat` | Build and test scripts for Windows |
| `project.bn` | Main sample program |
| `tests/` | Test programs (`01`-`12` correct, `w1` warnings, `e1`-`e4` errors) |

## Requirements

- **Flex**
- **GCC** (the generated C is compiled with `-std=c99`)
- Windows is the primary target (a UTF-8 capable terminal such as **Windows Terminal** is recommended for joined Bengali letters). The driver also has non-Windows paths, so Linux and macOS builds are possible with the manual command below.

## Build

**Windows** (CMD in the project folder):

```
build
```

This runs Flex and GCC and creates `bnc.exe`.

**Manual build** (any system with Flex and GCC):

```
flex project.l
gcc lex.yy.c parser.c ast.c symbol_table.c semantic.c tac.c code_generator.c main.c -o bnc
```

## Usage

```
bnc project.bn
```

Example output (typing `20` when asked):

```
Compiling project.bn

[1] Lexical analysis .... OK (75 tokens)
[2] Syntax analysis ..... OK (syntax tree built)
[3] Semantic analysis ... OK (3 variables, 0 warnings)
[4] Code generation ..... OK -> output.c
[5] GCC compilation ..... OK -> program.exe

========== PROGRAM OUTPUT ==========
বয়স লিখুন: 20
রহিম ভোট দিতে পারবে
1
2
3
========== END ==========
```

### Options

| Command | Shows |
|---------|-------|
| `bnc file.bn` | compile and run |
| `bnc file.bn --tokens` | token list (lexical analysis) |
| `bnc file.bn --ast` | syntax tree (syntax analysis) |
| `bnc file.bn --symbols` | symbol table (semantic analysis) |
| `bnc file.bn --tac` | three-address code |
| `bnc file.bn --all` | all four of the above |
| `bnc file.bn --no-run` | compile only, do not run |

To save the output of every phase and view it with correct Bengali rendering:

```
bnc project.bn --all > result.txt
code result.txt
```

## The language

### Keywords

| Bengali C | C | | Bengali C | C |
|-----------|---|-|-----------|---|
| `পূর্ণসংখ্যা` | `int` | | `করো` | `do` |
| `দশমিক` | `float` | | `থামো` | `break` |
| `অক্ষর` | `char` | | `পরবর্তী` | `continue` |
| `প্রধান` | `main` | | `ফেরত` | `return` |
| `যদি` | `if` | | `দেখাও` | `printf` |
| `নাহলে` | `else` | | `নাও` | `scanf` |
| `যতক্ষণ` | `while` | | `এবং` | `&&` |
| `গণনার জন্য` | `for` | | `অথবা` | `\|\|` |
| | | | `না` | `!` |

The symbols `&&`, `||` and `!` also work.

### Example program

```
পূর্ণসংখ্যা প্রধান()
{
    পূর্ণসংখ্যা নম্বর[৫] = {৮০, ৭৫, ৯০, ৬৫, ৮৫};
    পূর্ণসংখ্যা মোট = ০;
    অক্ষর নাম[৩০];

    দেখাও("আপনার নাম: ");
    নাও("%s", নাম);

    গণনার জন্য (পূর্ণসংখ্যা i = ০; i < ৫; i++)
        মোট += নম্বর[i];

    যদি (মোট / ৫ >= ৮০)
        দেখাও("%s, আপনার গ্রেড A+\n", নাম);
    নাহলে
        দেখাও("%s, গড় %.2f\n", নাম, (দশমিক) মোট / ৫);

    ফেরত ০;
}
```

### Supported

- **Types:** `পূর্ণসংখ্যা` (int), `দশমিক` (float), `অক্ষর` (char)
- **Variables:** several per declaration, declarations anywhere in a block (C99 style)
- **Arrays:** up to 4 dimensions, initializer lists, size taken from the values, `অক্ষর` arrays as strings
- **Statements:** `যদি` / `নাহলে`, `যতক্ষণ`, `করো ... যতক্ষণ`, `গণনার জন্য`, `থামো`, `পরবর্তী`, `ফেরত`, blocks, single-statement bodies
- **Input/output:** `দেখাও` (printf) and `নাও` (scanf) with C's format codes, flags, width and precision
- **Operators:** `+ - * / %`, `++ --`, `= += -= *= /= %=`, comparisons, `&& || !`, bitwise operators, `?:`, `,`, casts, `[ ]`, all with C's precedence
- **Numbers:** Bengali or English digits (`২৫` / `25`, `৩.১৪`)
- **Comments:** `//` and `/* */`

## Error handling

Messages show the line number and phase. A few real examples:

```
Line 4: Syntax error: expected ';' at the end of the declaration, but found 'দেখাও'
Line 6: Semantic error: variable 'উচ্চতা' is used but was never declared
Line 9: Semantic error: নাও: %d needs the address of the variable: write &বয়স
Line 5: Warning: '=' in the condition of যদি stores a value; to compare two values use '=='
```

| Mistake | Caught in |
|---------|-----------|
| unknown character, unclosed text or comment | Lexical analysis |
| missing `;` `(` `)` `{` `}`, misspelled keyword, missing `প্রধান()` | Syntax analysis |
| undeclared or duplicate variable, wrong format code, missing `&`, array misuse, `থামো` outside a loop | Semantic analysis |

## Testing

```
run_tests
run_tests > test_results.txt
```

| Group | Files | Expected |
|-------|-------|----------|
| Correct programs | `01_hello` to `12_scope` (variables, operators, input, if/else, loops, logic, arrays, primes, bubble sort, factorial/Fibonacci, scope) | compile and run |
| Warnings | `w1_warnings.bn` | warnings shown, program still runs |
| Errors | `e1_lexical_errors`, `e2_syntax_errors`, `e3_semantic_errors`, `e4_missing_main` | rejected with error messages |

`04_input.bn` reads its input from `tests/input.txt`.

## Limitations

Bengali C covers the C of a first programming course. Not included:

- functions other than `প্রধান()`, and global variables
- pointers (except `&` inside `নাও`), `struct`, `union`, `enum`
- `switch`, `goto`
- `double`, `long`, `short`, `unsigned`
- `#include`, `#define` and library functions such as `strlen`
- hexadecimal and octal numbers
- Bengali letters inside `অক্ষর` values (a Bengali letter takes 3 bytes in UTF-8)
- Output uses English digits, and typed input must use English digits

## Future improvements

User-defined functions, `switch`, `double`, string functions, printing Bengali digits, a Bison-based parser, and assembly generation from the three-address code.

## Built with

- C
- Flex
- GCC
