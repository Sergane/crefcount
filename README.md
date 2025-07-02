# ELF Symbol Cross-Reference Counter

This Python script analyzes ELF object files (`.o`) to count cross-references (like [this](https://maskray.me/blog/2022-02-27-analysis-and-introspection-options-in-linkers#cross-references)) between symbols by inspecting relocation entries. It helps understand symbol dependencies across different compilation units.

## Example Output

The script generates a JSON output that lists, for each object file:

1. Symbols **defined** in the object file.
2. Symbols **referenced** from other object files, along with their relocation count and source object(s).

```
/usr/lib/crt1.o
/usr/lib/crti.o
/usr/lib/gcc/x86_64-pc-linux-gnu/15.1.1/crtbeginT.o
CMakeFiles/main.dir/main.c.o
CMakeFiles/main.dir/a.c.o
libB.a
libC.a
/usr/lib/gcc/x86_64-pc-linux-gnu/15.1.1/libgcc.a
/usr/lib/gcc/x86_64-pc-linux-gnu/15.1.1/libgcc_eh.a
/usr/lib/libc.a
/usr/lib/gcc/x86_64-pc-linux-gnu/15.1.1/crtend.o
/usr/lib/crtn.o
/usr/lib/crtn.o [12 of 12] 100% |##########################################| Elapsed Time: 0:00:00 | Time:  0:00:00
```


```json
{
  ...
  "CMakeFiles/main.dir/main.c.o": [
    [
      "main"
    ],
    [
      {
        "name": "a_foo",
        "count": 2,
        "obj": [
          "CMakeFiles/main.dir/a.c.o"
        ]
      }
    ]
  ],
  "CMakeFiles/main.dir/a.c.o": [
    [
      "a_foo"
    ],
    [
      {
        "name": "b_foo",
        "count": 4,
        "obj": [
          "libB.a"
        ]
      }
    ]
  ],
  "libB.a": [
    [
      "b_foo"
    ],
    [
      {
        "name": "c_func",
        "count": 1,
        "obj": [
          "libC.a"
        ]
      }
    ]
  ],
  "libC.a": [
    [
      "c_func"
    ],
    []
  ],
  ...
}
```

- The first list contains **symbols defined** in that object file.
- The second list contains **symbols referenced** from elsewhere, including how many times they were used (`reloc_count`) and which object(s) originally defined them.

## Usage

```bash
crefcount [-h] [-d DEPFILE | -f FILE] [-C DIRECTORY] [-o OUT]

ELF Symbol Cross-Reference Counter

options:
  -h, --help            show this help message and exit
  -d, --depfile DEPFILE
                        path to depfile from linkers --dependency-file option
  -f, --file FILE       path to object file (ignores -o option)
  -C, --directory DIRECTORY
                        relative file paths are considered relative to this directory
  -o, --out OUT         path to output json file
```

Example:
```bash
cd examples/dummy
cmake -S . -B build && cmake --build build
crefcount -d build/CMakeFiles/main.dir/link.d
# or for one object only:
crefcount -f build/CMakeFiles/main.dir/main.c.o
```
See resulting [output json](examples/dummy/objects.json).

## Dependencies

- Python 3.x
- [`pyelftools`](https://github.com/eliben/pyelftools)

Install dependencies via pip:

```bash
# create virtualenv with given python version
export DEB_PYTHON_INSTALL_LAYOUT='deb' # optionally for Ubuntu
virtualenv -p python3.10 venv
# activate virtualenv
. venv/bin/activate
# install dependencies
pip install -r requirements.txt
```

## Features

- Analyze `.o` object files (`.a` static libraries in future - see TODO section)
- Count symbol references using relocation entries
- Generate structured JSON output for further processing

## TODO

- [ ] **Parse given headers to find symbol definitions with `libclang`**  
  Improve symbol detection by analyzing C/C++ headers using Clang AST parsing.

- [ ] **Add static libraries (`.a`) support**  
  Partially implemented (object counts instead of relocations) — need to fully support symbol extraction and relocation resolution from AR-archives.

## License

MIT License – see `LICENSE` for details.
