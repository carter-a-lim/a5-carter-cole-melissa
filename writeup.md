Assignment 5
=========

# Team Members:
- Carter
- Cole
- Melissa

# Expected Data Structures
If we were building a coverage tester, we would need a few main data structures. We would use dictionaries to connect each file name to the coverage information for that file. We would use sets of line numbers so we can quickly track which lines ran, without counting the same line over and over. For branch coverage, we would likely use pairs of line numbers (like from one line to the next) to show how the program moved through `if` and `loop` statements. We would also want a way to group results by context, such as by test name, so another dictionary would help there too. Because coverage results can get large, we would store data in a compact format on disk, likely in SQLite. Finally, we would use small lookup caches to avoid repeating expensive checks while the program is running.

# Initial Code Examination
The repository is organized clearly. At the top level, `coverage/` has the main program code, `tests/` has test files, and `doc/` has written documentation. There are also setup files like `pyproject.toml` and `tox.ini`. The README was helpful because it quickly explained what the project does, which Python versions it supports, and where the full docs are.

Using PowerShell, we counted about: `coverage/` 62 files, 20,407 lines, 71,161 words; `tests/` 199 files, 39,270 lines, 113,222 words; `doc/` 69 files, 23,141 lines, 130,020 words. This tells us a big part of the project is tests and docs, which usually means the code is taken seriously and checked carefully.

We randomly sampled these files:
- `tests/test_numbits.py`: tests line-number packing logic with many kinds of input and some database checks.
- `tests/test_cmdline.py`: checks that command-line options call the right actions.
- `coverage/numbits.py`: turns sets of line numbers into a compact byte format and back.
- `coverage/collector.py`: collects which lines run while code executes, including support for threads.
- `coverage/sqldata.py`: saves and reads coverage data from SQLite.
- `coverage/parser.py`: reads Python source and figures out which lines count as executable.
- `coverage/cmdline.py`: defines commands and options for running coverage from the terminal.

Overall, reading the code was the main way to understand behavior, but the comments were still useful when parts got tricky.

# Detailed Code Examination
For the detailed look, we focused on `coverage/jsonreport.py`, which builds the JSON output users see after running coverage. This file mostly uses simple dictionaries and lists.

The main structure is a dictionary named `report_data`. It starts with a `meta` section (version, timestamp, and options), then adds `files`, then `totals`. This makes the output easy to follow because each top-level key has one clear purpose.

Inside `report`, the code loops through files and builds another dictionary called `measured_files`. Each key is a file name, and each value is the result from `report_one_file`. That result is also a dictionary with basic fields like `executed_lines`, `missing_lines`, and `summary`. This is a good fit for JSON because dictionaries naturally map to JSON objects.

Lists are used for ordered coverage details. For example, executed and missing lines are stored as sorted lists so output is stable and easy to read. Branch data is also turned into a list of two-number pairs by `_convert_branch_arcs`, which makes it simple to serialize.

Another helpful pattern is that `make_summary` and `make_branch_summary` each build small dictionaries with percentages and counts. Those helper methods keep the code cleaner and avoid repeating the same key-building logic in many places.

Overall, the data flow is straightforward: collect analysis results, place them into nested dictionaries and lists, then call `json.dump`. We liked this file because it uses basic structures in a clear way and turns internal coverage data into a format that people and tools can read easily.

# Summary
This codebase feels high quality and easier to maintain than many student projects. Most names are clear, files are organized by purpose, and docstrings explain what functions are trying to do. Comments are used in a balanced way: not too many, but helpful when logic gets difficult.

Some parts still take time to understand, especially low-level tracing code, but strong tests make the project easier to trust. We also noticed good separation of responsibilities: command-line code, tracing, parsing, storage, and reporting are kept in different modules.

Compared to our own code, this project is more disciplined about unusual cases and long-term compatibility. The large test suite and documentation make us more confident about making changes. We would be comfortable maintaining it, but we would still be careful around tracing and data-storage code because small errors there could quietly produce wrong coverage results.