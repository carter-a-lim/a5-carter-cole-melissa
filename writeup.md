Assignment 5
=========

# Team Members:
- Carter
- Cole
- Melissa

# Expected Data Structures
If we were building a coverage tester, here are the main data structures we would use. A `dict[str, FileCoverage]` would map each file name to its coverage data — O(1) lookup, like a hash table. Each file's result would be a frozen `@dataclass` (a product type) with fields like `executed_lines: List[int]` and `missing_lines: List[int]`, stored as sorted lists so output stays consistent. For branch coverage, we would add a `List[tuple[int, int]]` where each pair like `(5, 10)` means the program jumped from line 5 to line 10 through an `if` or loop. Another `dict[str, FileCoverage]` would group results by test name, so we can look up which test caused which lines to run. We would use frozen dataclasses throughout since we are doing functional-style programming — the goal is to make new data rather than change existing data. Results would be saved to a file on since keeping everything in memory would be too large.

# Initial Code Examination
The repository is organized clearly: `coverage/` has the main source code, `tests/` has test files, and `doc/` has documentation. The README was helpful — it explained what the project does, which Python versions it supports, and where the full docs are.

Using PowerShell, we counted: `coverage/` ~62 files, 20,407 lines; `tests/` ~199 files, 39,270 lines; `doc/` ~69 files, 23,141 lines. Having `tests/` be larger than `coverage/` shows the project takes testing seriously.

We randomly sampled these files:
- `tests/test_numbits.py`: Tests line-number packing with many kinds of input. Uses `assertEqual` calls just like our `unittest.TestCase` pattern.
- `tests/test_cmdline.py`: Checks that command-line options trigger the right functions. Mostly straightforward assertions.
- `coverage/numbits.py`: Converts lists of line numbers into a compact byte format and back. Functions follow the underscore naming convention.
- `coverage/collector.py`: Tracks which lines run while code executes. More complex due to thread support.
- `coverage/sqldata.py`: Saves and reads coverage data from a database file on disk. The largest file in the project.
- `coverage/parser.py`: Reads Python source and determines which lines are executable. Heavy use of sets and dicts internally.
- `coverage/cmdline.py`: Defines the commands and flags for running coverage from the terminal.

For all files, reading the code was the main way to understand behavior. Comments were helpful when the logic got tricky.

# Detailed Code Examination
We focused on `coverage/jsonreport.py`, which builds the JSON output after running coverage. It mostly uses `dict` and `list` — the same structures from class.

The top-level structure is a `dict` called `report_data` with three keys: `meta` (version, timestamp, options), `files`, and `totals`. This is a product type — each key holds one specific piece of data, and together they fully describe a coverage run.

Inside `report()`, the code loops over every measured file — O(n) — and builds a `dict[str, dict]` called `measured_files`. Each key is a file name string and each value comes from `report_one_file()`. That value is another `dict` with fields like `executed_lines`, `missing_lines`, and `summary`. It behaves like a frozen `@dataclass` but is stored as a plain `dict` so it can be serialized directly to JSON.

`executed_lines` and `missing_lines` are `List[int]`, sorted so output is always in the same order regardless of which tests ran first. Branch data comes out of `_convert_branch_arcs()` as a `List[tuple[int, int]]` — each pair `(from_line, to_line)` is a product type of two ints representing one possible path through an `if` or loop.

The helper functions `make_summary()` and `make_branch_summary()` each build and return small `dict`s with counts and percentages. Separating them out follows the design recipe idea that each function should do one thing — it also avoids repeating the same key-building logic in multiple places.

Data flows in one direction: collect analysis results, place them into nested dicts and lists, then call `json.dump()`. All lookups inside the loop are O(1) since they use dicts. The function names (`report_one_file`, `make_summary`, `_convert_branch_arcs`) all follow the underscore naming convention and clearly state their purpose.

# Summary
This codebase is higher quality than most student projects. Functions use underscore names, types use CamelCase, files are split by purpose, and docstrings explain what each function does — basically the design recipe applied at scale.

The test suite uses `unittest.TestCase` the same way we do in class. Having `tests/` be larger than `coverage/` shows edge cases are taken seriously, and the tests make the project safer to change.

Some parts are still hard to follow, especially the low-level tracing code. It has side effects and more moving parts, which makes it harder to reason about than a pure function. We would be careful there.

Compared to our own code, this project is more consistent about separating concerns — tracing, parsing, storage, and reporting are all in different modules, each doing one thing. We would feel comfortable working in most of it, but would be cautious around the tracing and database code since small bugs there could produce silently wrong coverage results.