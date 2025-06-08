# Chapter 1: Getting Started with Typocalypse

Welcome to the very first chapter of our journey: building **Typocalypse**, a minimalist, yet powerful text editor in C. In this chapter, we’ll set up our Linux development environment, compile our first stub program, and lay down the theoretical foundation that will power all the raw-mode terminal magic to come.

## Why "Typocalypse"?

The name **Typocalypse** is a playful mash-up of **"type"** and **"apocalypse"**—evoking the idea of unleashing a storm of characters with every keystroke! As a text editor, Typocalypse aims to give you direct, low-level control over terminal input and output, so you can sculpt your text world precisely as you see fit.

---

## What Does a Text Editor Actually Do?

Before diving into implementation details, it’s helpful to break down the core responsibilities of a text editor—especially one running in a terminal:

1. **Display Buffer Management**

   * Maintain an in-memory array (or similar structure) of lines and characters.
   * Efficiently insert, delete, and modify text without redrawing the entire screen every time.

2. **User Input Handling**

   * Read keypress events one at a time (hence raw mode).
   * Distinguish printable characters, control keys (arrows, Home/End), and command shortcuts (e.g., Ctrl-S to save).

3. **Cursor Movement and Screen Refresh**

   * Track the cursor’s logical position within the buffer.
   * Map that to a physical position on the screen, handling line wrapping and scrolling.
   * Refresh only the parts of the screen that changed, minimizing flicker.

4. **File I/O**

   * Load existing text files into the buffer at startup.
   * Write the buffer back to disk on save, handling errors gracefully (e.g., disk full).

5. **Editing Commands and Modes**

   * Provide basic commands: insert text, delete text, cut/copy/paste.
   * Optionally support modes (e.g., insert vs. command mode) or shortcuts for power users.

6. **Syntax and Search (Advanced)**

   * Highlight syntax based on file type.
   * Search within the buffer, supporting regular expressions or incremental search.

### How to Think Like an Editor-Builder

* **Start Small**: Focus on displaying static text and moving the cursor around.
* **Iterate Quickly**: Build one feature at a time—raw-mode input, then cursor movement, then text insertion.
* **Data Structures Matter**: Choose a buffer representation that makes common operations (like insertion) efficient.
* **Optimize Only When Necessary**: Begin with a simple array-of-strings, and refactor to a gap buffer or rope if performance demands it.
* **Embrace Terminal Controls**: Learn ANSI escape codes for cursor movement, color, and screen clearing. These are your painting tools.

With this mental model in place, you’ll know exactly where to focus in each chapter. Now, let’s see our high‑level plan for Chapter 1!

## Today’s Roadmap

1. **Theory Spotlight**: Understand how terminals process input and why we need "raw mode."
2. **Environment Setup**: Verify and install `gcc`/`cc` and `make` on Linux.
3. **Hello, Typocalypse**: Write `typocalypse.c` with a minimal `main()` stub.
4. **Automate with Make**: Create a Makefile to compile into an executable named `main`.
5. **Test Drive**: Compile, run, and inspect the exit code to confirm success.

By the end of this chapter, our build system will be humming, and you’ll have the theoretical insight needed to tackle raw-mode terminal I/O in Chapter 2.

---

## 1. Theory Spotlight: Terminals, Canonical vs. Raw Mode

Most shells operate in **canonical mode**, where your input is buffered and line‑edited—press Backspace to erase, and only when you hit Enter does the program see your text. But a text editor needs to respond instantly to each keypress: arrow keys, Ctrl-K, even Ctrl-C must be handled in real time.

To break free from the shell’s line discipline, we switch the terminal into **raw mode**. In raw mode:

* **No buffering**: Input appears character-by-character.
* **No echo**: Keystrokes aren’t automatically printed.
* **No special handling**: Ctrl-C, Ctrl-Z, and others send their raw byte codes to your program instead of triggering signals.

We’ll achieve this by tweaking the terminal flags defined in `<termios.h>`:

* **ECHO**: whether typed characters are shown.
* **ICANON**: whether input is buffered until newline.
* **ISIG**: whether special characters generate signals.
* **IXON**: whether Ctrl-S/Ctrl-Q (XON/XOFF) flow control is enabled.

**Analogy**: Think of canonical mode as a polite tea‑party host who waits until everyone’s finished speaking (Enter) before summarizing (delivering input). Raw mode is more like a live podcast—every whisper, cough, and click goes straight to the mics.

---

## 2. Environment Setup

Our code will use `<termios.h>`, which isn’t available on Windows. Here’s how you prepare on Linux:

1. **Install the C Compiler and Make**

   ```bash
   sudo apt update
   sudo apt install gcc make
   ```

   This installs the GNU Compiler Collection (`gcc`) and the `make` utility.

2. **Verify the installation**

   ```bash
   cc --version    # should print the compiler version
   make -v         # should print the make version
   ```

   If you see version information, you’re all set!

---

## 3. Writing the First Source File

1. Create `typocalypse.c`:

   ```bash
   touch typocalypse.c
   ```
2. Open `typocalypse.c` and add:

   ```c
   int main() {
       return 0;
   }
   ```

> **Why ************\`\`************?\*\*\*\*\*\*\*?**
> Execution in C begins at `main()`. Returning `0` tells the OS everything went smoothly.

---

## 4. Creating the Makefile

Forget typing the compile command by hand—let Make do it.

Create a file named `Makefile` and add (use a real **tab** before the compile line):

```makefile
# Makefile for Typocalypse

main: typocalypse.c
	$(CC) typocalypse.c -o main -Wall -Wextra -pedantic -std=c99
```

* \`\` defaults to `cc`.
* \`\` enable helpful warnings.
* \`\` locks the C99 standard.
* \`\` is our output executable, replacing the source name.

---

## 5. Compiling and Running

1. **Build**

   ```bash
   make
   ```

   * If nothing has changed, you’ll see `make: 'main' is up to date.`
   * If you edited `typocalypse.c`, Make rebuilds `main`.

2. **Run**

   ```bash
   ./main
   ```

   No output yet—this is expected.

3. **Check the exit status**

   ```bash
   echo $?
   ```

   You should see `0`, confirming your program returned success.

> **Pro tip:** Change `return 0;` to `return 5;`, re-run `make`, `./main`, and `echo $?`. You’ll observe the exit code `5` show up!

---

## Looking Ahead

Armed with environment setup and terminal theory, you’re ready to plunge into **Chapter 2**. We’ll:

* Switch the terminal into **raw mode** using `<termios.h>`.
* Read individual keystrokes without waiting for Enter.
* Start painting characters and controlling cursor movement.

Brace yourself: the keyboard is about to become your paintbrush! 🎨

