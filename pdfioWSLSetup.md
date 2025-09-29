# Building the PDFio Library on WSL (Ubuntu)

This guide provides a straightforward, step-by-step process for compiling and installing the `pdfio` C library from source on a Windows Subsystem for Linux (WSL) Ubuntu environment. Following these steps will prepare your system for developing applications that use `pdfio`.

## Prerequisites

Before you begin, ensure you have the following:

  * A working WSL 2 instance installed on your Windows machine.
  * An Ubuntu distribution installed from the Microsoft Store.

-----

### Step 1: Install Build Dependencies 🛠️

First, you need to update your system's package list and install the essential tools required for compiling C programs. This includes the `build-essential` package (which contains GCC, `make`, etc.), `git` for downloading the source code, and the `zlib` development library, which is a required dependency for `pdfio`.

```bash
sudo apt update && sudo apt install -y build-essential zlib1g-dev git
```

  * **Explanation**: This command ensures your Ubuntu environment has all the necessary prerequisites for the compilation process.

-----

### Step 2: Download the Source Code 📥

Next, use `git` to clone the official `pdfio` repository from GitHub. This will download the latest source code into a new directory named `pdfio`.

```bash
git clone https://github.com/michaelrsweet/pdfio.git
cd pdfio
```

  * **Explanation**: This downloads a local copy of the project and moves your terminal into the project's root directory.

-----

### Step 3: Configure the Build ⚙️

Inside the `pdfio` directory, run the `configure` script. This script checks your system to ensure all dependencies are present and creates a `Makefile` that is customized for your specific environment.

```bash
./configure --enable-shared
```

  * **Explanation**: We use the `--enable-shared` flag to instruct the build system to create a shared library (`.so` file) in addition to the default static library (`.a` file). Shared libraries are often more flexible for development.

-----

### Step 4: Compile the Library 🧱

With the `Makefile` successfully generated, you can now compile the entire library's source code by running the `make` command.

```bash
make all
```

  * **Explanation**: This command reads the `Makefile` and executes the necessary compilation steps to build the `pdfio` library from the source files.

-----

### Step 5: Run Verification Tests ✅

Before installing the library system-wide, it is highly recommended to run the built-in tests. This step verifies that the library was compiled correctly and functions as expected on your system.

```bash
make test
```

  * **Explanation**: This executes a series of automated checks against the newly compiled library to ensure its integrity and functionality.

-----

### Step 6: Install the Library 🚀

Finally, install the compiled library and its header files to the standard system directories. This makes the library available for any C project on your system to use.

```bash
sudo make install
```

  * **Explanation**: This command copies the compiled library files and headers to locations like `/usr/local/lib/` and `/usr/local/include/`. `sudo` is required as this is a system-wide change.

-----

## Verifying the Installation

You can quickly verify that the library was installed correctly by listing the installed files.

```bash
ls -l /usr/local/lib/libpdfio.*
```

You should see output showing the newly created library files, such as `libpdfio.so` and `libpdfio.a`.

## Next Steps: Using the Library

The `pdfio` library is now ready to be used in your C projects. When compiling a program that uses it, you will need to:

1.  Include the main header file in your C code:

    ```c
    #include <pdfio.h>
    ```

2.  Link against the library during compilation using the `-lpdfio` flag:

    ```bash
    gcc your_program.c -o your_program -lpdfio
    ```
