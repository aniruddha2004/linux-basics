# Archiving and Compression

> Quick notes on packing files together (archiving) and making them smaller (compression).

---

## Archiving vs Compression

| | Archiving | Compression |
|--|-----------|-------------|
| **What it does** | Combines multiple files/directories into one file | Reduces the size of a file |
| **Example tool** | `tar` | `gzip`, `bzip2`, `xz`, `zip` |
| **Goal** | Organization and easy transport | Save disk space and bandwidth |
| **Analogy** | Putting files into a suitcase | Vacuum-sealing the suitcase to make it smaller |

- **Archive** = a collection of files bundled together (`.tar`)
- **Compressed archive** = a bundled collection that has also been shrunk (`.tar.gz`, `.zip`)

---

## `tar` Commands

### Create a Compressed Archive

```bash
tar -zcvf demo.tar.gz folder_name
```

| Flag | Meaning |
|------|---------|
| `-z` | Compress with gzip |
| `-c` | Create archive |
| `-v` | Verbose (show files being processed) |
| `-f` | Specify filename |

---

### Extract a Compressed Archive

```bash
tar -zxvf demo.tar.gz
```

| Flag | Meaning |
|------|---------|
| `-z` | Decompress with gzip |
| `-x` | Extract |
| `-v` | Verbose |
| `-f` | Specify filename |

---

### Extract to a Specific Directory

```bash
tar -zxvf demo.tar.gz -C /path/to/destination
```

---

## Using `tar` and `gzip` Separately

You can do the two steps manually instead of combining them with `tar -z`.

### Step 1: Create Archive (No Compression)

```bash
tar -cvf demo.tar folder_name
```

Creates `demo.tar` — an uncompressed bundle.

### Step 2: Compress the Archive

```bash
gzip demo.tar
```

Produces `demo.tar.gz` and deletes the original `demo.tar`.

### Step 3: Decompress

```bash
gunzip demo.tar.gz
```

Restores `demo.tar`.

### Step 4: Extract

```bash
tar -xvf demo.tar
```

---

### Separate-Tool Equivalent

| Combined | Separate |
|----------|----------|
| `tar -zcvf demo.tar.gz folder_name` | `tar -cvf demo.tar folder_name` then `gzip demo.tar` |
| `tar -zxvf demo.tar.gz` | `gunzip demo.tar.gz` then `tar -xvf demo.tar` |

---

## Common Extensions

| Extension | Meaning |
|-----------|---------|
| `.tar` | Plain archive |
| `.tar.gz` or `.tgz` | Archive compressed with gzip |
| `.tar.bz2` | Archive compressed with bzip2 |
| `.tar.xz` | Archive compressed with xz |
| `.zip` | Zip archive (compression built-in) |

---

## Brief Note: Tar Bombs

A **tar bomb** is an archive that extracts all its files directly into the current directory instead of into a single containing folder.

Example:

```bash
tar -zxvf messy.tar.gz
# Result: hundreds of files scattered in your current directory
```

### How to Avoid

- Always check archive contents first:

```bash
tar -ztvf demo.tar.gz
```

- Extract into a dedicated directory:

```bash
mkdir extracted
tar -zxvf demo.tar.gz -C extracted/
```

- When creating archives, include files inside one parent directory so extraction is clean:

```bash
tar -zcvf demo.tar.gz demo_folder/
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Create compressed archive | `tar -zcvf name.tar.gz folder/` |
| Extract compressed archive | `tar -zxvf name.tar.gz` |
| Extract to specific folder | `tar -zxvf name.tar.gz -C /destination` |
| View contents | `tar -ztvf name.tar.gz` |
| Create plain archive | `tar -cvf name.tar folder/` |
| Compress with gzip | `gzip name.tar` |
| Decompress with gzip | `gunzip name.tar.gz` |
