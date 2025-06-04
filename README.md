# PP7

## Goal

In this exercise you will:

* **Understand** each stage of the C compilation pipeline (preprocessing, compilation to assembly, assembling, and linking).
* **Inspect** intermediate outputs to see how your C code transforms at each step.
* **Use** text‑processing tools (`grep`, `sed`, `awk`) with regular expressions to search and replace patterns in code.
* **Automate** search/replace interactively and non‑interactively in **Vim**.
* **Modularize** C code by defining functions across separate files and **link** them together manually.

**Important:** Start a stopwatch when you begin and work uninterruptedly for **90 minutes**. When time is up, stop immediately and record exactly where you paused.

---

## Workflow

1. **Fork** this repository on GitHub.
2. **Clone** your fork to your local machine.
3. Create a directory at the repository root named `solutions/`:

   ```bash
   mkdir solutions
   ```
4. For each task below, add the required files under `solutions/` (for example, `solutions/sample.c`, `solutions/sample.i`, etc.).
5. **Commit** your changes locally, then **push** them to GitHub.
6. **Submit** your repository link for review.

---

## Prerequisites

* A C compiler toolchain (e.g. `gcc`).
* Familiarity with command‑line tools: `grep`, `sed`, `awk`, and `vim`.
* Man‑pages for reference:

  ```bash
  man gcc   # for compiler options
  man grep  # for searching
  man sed   # for stream editing
  man awk   # for pattern scanning
  man vim   # for interactive editing
  ```

---

## Tasks

### Task 1: C Compilation Stages

**Objective:** Break down the process of transforming C source into an executable and inspect each intermediate file.

1. In `solutions/`, create a C source file named `sample.c` containing a simple `main()`:

   ```c
   #include <stdio.h>

   int main(void) {
       printf("Hello, PP7!\n");
       return 0;
   }
   ```
2. **Preprocess** only (run the preprocessor, `-E`) and save the output:

   ```bash
   gcc -E sample.c -o solutions/sample.i
   ```

   * Open `solutions/sample.i` in an editor and note how `#include <stdio.h>` expands and macros are handled.
```bash
-#include <stdio.h> is replace by the content and headers of stdio.h.
-The Line directives (e.g # 540 "/usr/include/stdio.h" 3 4
extern int vfscanf (FILE *__restrict __s, const char *__restrict __format, __gnuc_va_list __arg) __asm__ ("" "__isoc99_vfscanf")) tell the compiler where the code came from (important for error reporting and debugging)
-A lot of "extern" lines for standard library functions (e.g printf), commone types (e.g size_t), and structure definitions
```

3. **Compile to assembly** (`-S`):

   ```bash
   gcc -S solutions/sample.i -o solutions/sample.s
   ```

   * Examine `solutions/sample.s` to see the generated assembly instructions for `printf` and `return`.
```bash

**printf**

-"leaq	.LC0(%rip), %rax" is the core instruction for printf
 -leaq (load effective address): computes an address and loads it into a register.
 -.LC0(%rip): Calculates the address of the .LC0 label where the string "Hello, PP7!" is to the instruction      pointer "%rip". The return value is stored in %rax.
-"movq %rax, %rdi": moves the the data from the rax register to the rdi register.
- "call	puts@PLT": It calls the printf function or because of the linux system puts function. Through the dynamic linker the memory address of printf in libc and patches the PLT entry

**return 0**
-"movl	$0, %eax": The integer value "0" moves to the %eax register
-"popq %rbp": restores the base pointer and cleans up the stack frame, so that the program returns to the correct execution. 


```
```bash
   	.file	"sample.c"
	.text
	.section	.rodata
.LC0:
	.string	"Hello, PP7!"
	.text
	.globl	main
	.type	main, @function
main:
.LFB0:
	.cfi_startproc
	endbr64
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	leaq	.LC0(%rip), %rax
	movq	%rax, %rdi
	call	puts@PLT
	movl	$0, %eax
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	main, .-main
	.ident	"GCC: (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0"
	.section	.note.GNU-stack,"",@progbits
	.section	.note.gnu.property,"a"
	.align 8
	.long	1f - 0f
	.long	4f - 1f
	.long	5
0:
	.string	"GNU"
1:
	.align 8
	.long	0xc0000002
	.long	3f - 2f
2:
	.long	0x3
3:
	.align 8
4:
```

4. **Assemble** (`-c`):

   ```bash
   gcc -c solutions/sample.s -o solutions/sample.o
   ```

   * Verify that `solutions/sample.o` is an object file (e.g., with `file sample.o`).
```bash
The Output is: "sample.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped"
ELF (Executable and Linkable Format) is the standard binary file format. The file is a 64-bit architecture. It is a relocatble file, which means it is suitable for linking with other object files and libraries to create an executable. This confirms that solutions/sample.o is a compiled object file. 



```
5. **Link** to produce an executable:

   ```bash
   gcc solutions/sample.o -o solutions/sample
   ```

   * Run `./solutions/sample` and confirm it prints `Hello, PP7!`.
6. **Explain** in comments or a short README how each stage transforms the code.

---

### Task 2: Regex Search & Replace in Code

**Objective:** Use `grep`, `sed`, and `awk` to locate and modify code patterns, then perform the same edits in **Vim** both interactively and via the CLI.

1. Copy `solutions/sample.c` to `solutions/debug_sample.c`:

   ```bash
   cp solutions/sample.c solutions/debug_sample.c
   ```
2. **Search** for all calls to `printf` using `grep` with a regex pattern and save the results:

   ```bash
   grep -En "printf\s*\(" solutions/debug_sample.c
   ```
3. **Replace** each `printf` with `debug_printf` in the file using `sed` (in‑place):

   ```bash
   sed -E -i.bak 's/printf\s*/debug_printf/g' solutions/debug_sample.c
   ```
4. **Extract** lines containing `debug_printf` using `awk`:

   ```bash
   awk '/debug_printf/ { print NR, $0 }' solutions/debug_sample.c
   ```
5. **Vim Interactive**: open `solutions/debug_sample.c` in Vim:

   ```bash
   vim solutions/debug_sample.c
   ```

   * Use search (`/printf`) and substitute (`:%s/printf/debug_printf/g`) commands interactively.
   * Save and quit.
6. **Vim CLI**: perform the same substitute without opening the full UI:

   ```bash
   vim -c ":%s/printf/debug_printf/g" -c ":wq" solutions/debug_sample.c
   ```
7. **Explain** each tool’s approach to regex-based search and replace, and when you might prefer one over the others.

```
Grep (Global regular expression print)
command example: "grep -En "printf\s*\(" solutions/debug_sample.c"
Grep (Global regular expression print) searches the input line an prints the line matched to the input.

-E: It enables extend regular expression. It has a better syntax, which intrepreted characters like "()" ,"{}", "+" not only as literal character but as operator character.
n: It prints the lines where the searched inputs are.
\s*: Matches whitespace characters
\(: Is the opening parenthises followed the file name where the input should be searched ("solutions/debug_sample.c)

Sed (stream editor)
command example: "sed -E -i.bak 's/printf\s*/debug_printf/g' solutions/debug_sample.c"
Ser is a stream editor that is used to perform basic text transformation on an input stream line by line. 
It filters and transform text. Sed reads the input line by line and adds a set of editing commands to each line.
After that it write the result back to the file.

-E: It enables extend regular expression. It has a better syntax, which intrepreted characters like "()" ,"{}", "+" not only as literal character but as operator character.
-i.bak: It tells the sed to edit the file directly. Bak says the sed to create a backup copy of the original file before the change. The name of the bakup file is in this case "debug_sample.c.bak".
s: A substitute command to find and replace a text.
s/printf\s*/: The expression pattern to search each line with sed.
/printf\: The literal string.
\s*: Matches whitespace characters.
/debug_printf/: The string that replaces the literal string.
/g: A flag that that stands for global, which tells sed to replace all literal strings.

awk (Aho, Weiberger, Kernighan)
command example: "awk '/debug_printf/ { print NR, $0 }' solutions/debug_sample.c"
Awk is a pattern scanning and processing language. It looks for patterns and execute the 

Vim CLI
command example: "vim -c ":%s/printf/debug_printf/g" -c ":wq" solutions/debug_sample.c"

```


Sed (stream editor) can filter 
---

### Task 3: Modular Linking with `extern`

**Objective:** Create a separate addition function in `add.c`, declare it `extern` in `main.c`, and compile/link manually to form an executable.

1. In `solutions/`, create `add.c`:

   ```c
   // add.c
   int add(int a, int b) {
       return a + b;
   }
   ```
2. Create `main.c` that uses `add`:

   ```c
   // main.c
   #include <stdio.h>
   extern int add(int, int);

   int main(void) {
       int sum = add(5, 7);
       printf("5 + 7 = %d\n", sum);
       return 0;
   }
   ```
3. **Compile** each source separately:

   ```bash
   gcc -c solutions/add.c -o solutions/add.o
   gcc -c solutions/main.c -o solutions/main.o
   ```
4. **Link** the object files manually:

   ```bash
   gcc solutions/add.o solutions/main.o -o solutions/add_example
   ```
5. Run `./solutions/add_example` to verify it prints `5 + 7 = 12`.
6. **Document** the workflow in comments or a short file, emphasizing:

   * The role of `extern` declarations.
   * Why separating compilation can speed up builds.
   * How manual linking differs from letting `gcc` handle all steps in one command.

---

**Remember:** Stop working after **90 minutes** and record where you stopped.
