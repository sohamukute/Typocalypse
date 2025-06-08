**Linux-Only Setup Guide: Getting Started with C and Make**

This guide covers all the steps you need on a Linux system. We’ll keep it simple:

* A C compiler (`gcc` or `cc`)
* The `make` build tool

---

## 1. Check Your Tools

Open a terminal and run:

```bash
cc --version    # Check for a C compiler
make -v         # Check for make
```

If you see version information for both, you’re ready. Otherwise, install them in the next step.

---

## 2. Install on Linux

Most Linux distributions provide `gcc` and `make` in their package managers.

### Ubuntu, Debian, and derivatives

```bash
sudo apt-get update
sudo apt-get install gcc make
```

### Fedora, CentOS, RHEL

```bash
sudo dnf install gcc make
# or on older CentOS/RHEL: sudo yum install gcc make
```

### Arch Linux and derivatives

```bash
sudo pacman -Sy gcc make
```

If you use another distribution, look for `gcc` and `make` in your package manager.

---

## 3. Create Your First C Program

1. Choose a folder and create a file named `typocalypse.c`:

   ```bash
   nano typocalypse.c
   ```
2. Paste this code:

   ```c
   // typocalypse.c
   int main() {
     return 0;
   }
   ```
3. Save and exit (in `nano`, press `Ctrl+O`, `Enter`, then `Ctrl+X`).

---

## 4. Compile and Run Manually

1. **Compile**:

   ```bash
   cc typocalypse.c -o main
   ```

   * `-o main` names the output program `main`.
2. **Run**:

   ```bash
   ./main
   ```
3. Check the exit status:

   ```bash
   echo $?    # Should print 0 for success
   ```

---

## 5. Automate with a Makefile

Typing the compile command repeatedly is tedious. Let’s use **Make**.

1. In the same folder, create a file named `Makefile` (no extension).
2. Add these lines, using a real **tab** before the compile command:

   ```makefile
   # Makefile for typocalypse
   main: typocalypse.c
   	$(CC) typocalypse.c -o main -Wall -Wextra -pedantic -std=c99
   ```
3. Save the file.
4. Run:

   ```bash
   make
   ```

   * On first run, it compiles.
   * On subsequent runs, if `typocalypse.c` hasn’t changed, it reports `main is up to date.`

### Explanation of flags:

* `$(CC)` → the C compiler (`cc` by default)
* `-Wall` → enable common warnings
* `-Wextra` → enable extra warnings
* `-pedantic` → strictly follow the C standard
* `-std=c99` → compile in C99 mode

---

## 6. Test Code Changes

1. Edit `typocalypse.c` and change `return 0;` to another number (e.g., `return 5;`).
2. Run `make` again. You should see compilation happen.
3. Run `./main` and then `echo $?`. You’ll see the number you returned.
4. Change it back to `return 0;`, re-run, and confirm it prints 0.

**Tip:** Always run `make` before `./main` to ensure you’re executing the latest code.

---

Next, we’ll dive into reading keypresses directly and controlling your terminal. Happy coding!
