## Chapter 2: Entering Raw Mode

Welcome to one of the most magical moments in terminal programming—unlocking **raw mode**. If you're building something like a text editor, a terminal-based game, or just want to handle each keystroke yourself (and not wait until the user presses Enter), then raw mode is your backstage pass to complete control.

In this chapter, we'll learn how terminals behave, why they seem to buffer input, and how to override that behavior to make your own keystroke-driven interface. Along the way, we'll explore how to stop echoes, reclaim special keys, and gently return everything back to normal when you're done. Think of this chapter as your first deep dive into Unix terminal wizardry—explained with care.

### 🛠 What We're About to Do

Here's the journey ahead:

1. Read one keystroke at a time.
2. Disable automatic echoing by the terminal.
3. Save and restore the original terminal configuration.
4. Disable line buffering (canonical mode).
5. Handle `Ctrl-C`, `Ctrl-Z`, and other signal-generating keys yourself.
6. Turn off flow control that may freeze your program.
7. Make `Ctrl-V` behave like a regular key.
8. Handle carriage returns manually for line clarity.
9. Turn off output processing.
10. Wrap it all up and add a timeout for responsive input.

Each step is essential, and we’ll build it up piece by piece—explaining why we’re doing it, what flags or bits are being changed, and what effect each has.

---

### 1. Reading Keypresses: The Starting Point

```c
#include <unistd.h>
int main() {
  char c;
  // Keep reading until 'q' shows up
  while (read(STDIN_FILENO, &c, 1) == 1 && c != 'q');
  return 0;
}
```

> **What’s happening here?**
> This loop tries to read a single byte from standard input and continues until you type `q`. But you’ll notice something odd: you must hit **Enter** after each keystroke. That’s because the terminal is still in **canonical mode**, buffering input until you hit Enter.

Let’s change that.

---

### 2. Turning Off Echo

```c
#include <termios.h>
#include <unistd.h>

void enableRawMode() {
  struct termios raw;
  tcgetattr(STDIN_FILENO, &raw);            // Fetch current settings
  raw.c_lflag &= ~ECHO;                     // Clear the ECHO bit
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw); // Apply changes immediately
}
```

> **Why this matters:**
> Normally, when you type, the terminal echoes your input (so it appears on the screen). But if you're building a UI where **you** want to control what's printed (like a text editor), you don't want that echo. Disabling `ECHO` gives your program total control over the display.

---

### 3. Saving and Restoring Terminal Settings

```c
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>

struct termios orig_termios;

void disableRawMode() {
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &orig_termios); // Restore terminal
}

void enableRawMode() {
  tcgetattr(STDIN_FILENO, &orig_termios);  // Save original state
  atexit(disableRawMode);                  // Register cleanup at exit

  struct termios raw = orig_termios;
  raw.c_lflag &= ~ECHO;
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}
```

> **Why this is critical:**
> If you forget to restore the terminal’s original state, your shell could be left in a weird state—no echo, no line editing—until you reset it manually or reboot. `atexit()` ensures you always clean up, even if your program crashes.

---

### 4. Disabling Canonical Mode

```c
raw.c_lflag &= ~(ECHO | ICANON);
```

> **Explanation:**
> `ICANON` controls whether input is **line-buffered**. By disabling it, we tell the terminal: "send each keystroke right away." Now, `read()` gives you characters immediately—no need for Enter.

---

### 5. Taking Control of Signal Keys

```c
raw.c_lflag &= ~(ECHO | ICANON | ISIG);
```

> **What this does:**
> Signal-generating keys like `Ctrl-C` (SIGINT) and `Ctrl-Z` (SIGTSTP) are now just regular input bytes (3 and 26). This lets your program define its own behavior instead of being killed or suspended.

---

### 6. Turning Off Flow Control

```c
raw.c_iflag &= ~IXON;
```

> **Why turn off ****`IXON`****?**
> Legacy terminals pause/resume output with `Ctrl-S`/`Ctrl-Q`. In raw mode, that behavior can be frustrating. Disabling `IXON` ensures your UI never suddenly "freezes."

---

### 7. Making `Ctrl-V` a Regular Key

```c
raw.c_lflag &= ~(IEXTEN | ECHO | ICANON | ISIG);
```

> **What's ****`IEXTEN`****?**
> Extended input processing. On some systems, `Ctrl-V` tells the terminal to "quote" the next character. Clearing `IEXTEN` disables that, so `Ctrl-V` acts like a regular key (ASCII 22).

---

### 8. Handling Carriage Returns Manually

```c
raw.c_iflag &= ~ICRNL;

// When printing:
printf("%d ('%c')\r\n", c, c);
```

> **Why?**
> Terminals often auto-convert `\r` (carriage return) into `\n` (newline). Turning off `ICRNL` lets you detect carriage returns and handle line breaks yourself, using `\r\n` if needed.

---

### 9. Turning Off Output Post-Processing

```c
raw.c_oflag &= ~OPOST;
```

> **Impact:**
> Output like `\n` is normally translated to `\r\n`. Disabling `OPOST` means what you write is what gets displayed—no auto-formatting.

---

### 10. Putting It All Together—with Timeout

```c
void enableRawMode() {
  tcgetattr(STDIN_FILENO, &orig_termios);
  atexit(disableRawMode);

  struct termios raw = orig_termios;

  raw.c_iflag &= ~(BRKINT | ICRNL | INPCK | ISTRIP | IXON);
  raw.c_oflag &= ~OPOST;
  raw.c_cflag |=  CS8;
  raw.c_lflag &= ~(ECHO | ICANON | IEXTEN | ISIG);
  raw.c_cc[VMIN] = 0;
  raw.c_cc[VTIME] = 1;  // 0.1 second timeout

  if (tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw) == -1)
    die("tcsetattr");
}
```

> **Summary:**

* We disable input/output processing, echo, line buffering, signals, flow control, and more.
* We ensure the program exits cleanly, restoring the terminal.
* We add a small timeout for `read()`—making the program responsive even when idle.

This is the full recipe for **raw mode**: a foundation to build custom UIs, editors, and real-time terminal apps.

🎉 Congratulations—you now have deep control over the terminal. You're ready to start making something truly interactive!

> [Further reading: termios deep dive](https://pubs.opengroup.org/onlinepubs/009695399/functions/termios.html)
