# ELF Symbol Cross-Reference Counter

This Python script analyzes ELF object files (`.o`) to count cross-references (like [this](https://maskray.me/blog/2022-02-27-analysis-and-introspection-options-in-linkers#cross-references)) between symbols by inspecting relocation entries. It helps understand symbol dependencies across different compilation units.

## Example Output

The script generates a JSON output that lists, for each object file:

1. Symbols **defined** in the object file.
2. Symbols **referenced** from other object files, along with their relocation count and source object(s).

```json
{
  "a.o": [
    ["a_foo", "a_bar", "a_baz"],
    [
      {"name": "b_foo", "reloc_count": 4, "obj": ["b.o"]}
    ]
  ],
  "b.o": [
    ["b_foo"],
    []
  ]
}
```

- The first list contains **symbols defined** in that object file.
- The second list contains **symbols referenced** from elsewhere, including how many times they were used (`reloc_count`) and which object(s) originally defined them.

## Usage

```bash
crefcount <path_to_elf_or_directory>
```

Example:
```bash
./crefcount a.o
# or with objects in directory dir
./crefcount path/to/dir
```

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
  Currently under development — need to fully support symbol extraction and relocation resolution from AR-archives.

## License

MIT License – see `LICENSE` for details.
