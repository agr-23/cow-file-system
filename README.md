# cow-file-system

A small copy-on-write file system library in C++. Every write creates a new version of the file, unchanged blocks are shared between versions, and any version can be restored.

## Context

Team project for the Operating Systems course at EAFIT, by Alejandro Garcés, Jean Carlo Londoño Ocampo and Daniel Zapata Acevedo.

## How it works

All data lives in one binary file split into 4 KB blocks. Four pieces sit on top of it:

- `BlockManager` allocates, frees, reads and writes blocks, and saves the block map next to the data file.
- `Metadata` keeps, per file, its size, type and list of versions.
- `VersionGraph` stores each version with its block list, the blocks it changed and its parent version.
- `FileSystem` is the public API: `create`, `open`, `write`, `read`, `rollbackFile`, `close`, `listFiles`, `inspectBlocks`, `sync` and `printMemoryUsage`.

A `write` reads the current content, applies the change in memory and compares old and new data block by block. Only the blocks that changed get a new physical block. The rest of the new version points to the same blocks as its parent.

`VersionGraph::collectGarbage()` walks every file from its current version back to the root, marks the blocks those versions use and frees the rest. It is implemented but nothing calls it yet, so `sync()` only writes the block map and metadata to disk.

`bridge.cpp` exposes a C interface so the library can be loaded as a shared object, for example from Python with `ctypes`.

## Running it

Needs `g++` with C++20 and `make`.

```sh
make
make run      # runs the demo in main.cpp
make clean    # removes objects, the binary and the storage files
```

To build the shared library for the C bridge:

```sh
g++ -std=c++17 -fPIC -shared -o libcowfs.so bridge.cpp BlockManager.cpp FileSystem.cpp VersionGraph.cpp Metadata.cpp
```

Minimal use from C++:

```cpp
FileSystem fs("cow_data.bin", 20);   // 20 MB of storage
fs.create("foo.txt", "txt");
fs.open("foo.txt");
fs.write("foo.txt", 0, {'h', 'i'});
fs.rollbackFile("foo.txt", 1);
fs.close("foo.txt");
fs.sync();
```

## References

- simplyblock, "What is Copy-On-Write (CoW)?", YouTube, 2025.
- "Implementing copy-on-write", Software Engineering Stack Exchange, 2017.
- The original report also credits ChatGPT (GPT-4) and DeepSeek-V3 for conceptual help.

## License

No license has been chosen yet. Ask the authors before reusing the code.
