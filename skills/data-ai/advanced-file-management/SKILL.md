---
name: advanced-file-management
description: Advanced file management tools. Includes batch folder creation, batch file moving, file listing, and HTML author extraction.
---

# Advanced File Management Skill

This skill provides tools for desktop file management:

1. **Folder creation**: Create multiple folders under a target directory
2. **Move files**: Move multiple files to a target directory
3. **List all files**: Recursively list all files under a directory
4. **Extract authors**: Extract authors from HTML papers

## Important Notes

- **Do not use other bash commands**: Do not attempt to use general bash commands or shell operations like cat, ls.
- **Use relative paths**: Use paths relative to the working directory (e.g., `./folder/file.txt` or `folder/file.txt`).
- **Do not create scripts**: Do not use `write_file` to create any scripts.


## I. Skills

### 1. Create Multiple Folders

Creates multiple folders under a target directory.

### Features 

- Creates multiple folders with specified names
- Supports any number of folder names as arguments

### Example

```bash
# Create 3 folders
python create_folders.py . folder1 folder2 folder3

# Create folders with specific names
python create_folders.py ./projects experiments learning personal
```


### 2. Move Multiple Files

Move multiple files to a target directory in a single operation.

### Features

- Move multiple files at once
- Supports files from different directories
- Reports success and failure for each file

### Example

```bash
# Move 3 files to archive folder
python move_files.py ./archive file1.txt file2.txt file3.txt

# Move files from different directories
python move_files.py ./backup ./data/log1.txt ./data/log2.txt ./temp/cache.dat
```


### 3. List All Files

Recursively list all files under a given directory path. Useful for quickly understanding project directory structure.

### Features

- Recursively traverse all subdirectories
- Option to exclude hidden files (like .DS_Store)
- Output one file path per line, including both path and filename (relative to input directory)

### Example

```bash
# List all files (excluding hidden)
python list_all_files.py .

# Include hidden files
python list_all_files.py ./data --include-hidden
```

---

### 4. Extract Authors

Extract authors from all HTML papers in a directory using `<meta name="citation_author">` tags.

### Features

- Automatically scan all HTML files in directory
- Extract author names from citation_author meta tags
- Support multiple authors per paper
- Returns list of dicts with filename and authors

### Example

```bash
# Extract and print authors from all HTML files
python extract_authors.py ./papers

# Save to file
python extract_authors.py ./papers --output authors.txt
```

---

## II. Basic Tools (FileSystemTools)

Below are the basic tool functions. These are atomic operations for flexible combination.

**Prefer Skills over Basic Tools**: When a task matches one of the Skills above (e.g., creating multiple folders), use the corresponding Skill instead of Basic Tools. Skills are more efficient because they can perform batch operations in a single call.

**Prefer List All Files over list_directory/list_files**: When you need to list files in a directory, prefer using the `list_all_files.py` skill instead of `list_directory` or `list_files` basic tools. The skill provides recursive listing with better output formatting.

**Note**: Code should be written without line breaks.

### How to Run

```bash
# Standard format
python run_fs_ops.py -c "await fs.read_text_file('./file.txt')"
```

---

### File Reading Tools

#### `read_text_file(path, head=None, tail=None)`
**Use Cases**:
- Read complete file contents
- Read first N lines (head) or last N lines (tail)

**Example**:
```bash
python run_fs_ops.py -c "await fs.read_text_file('./data/file.txt')"
```

---

#### `read_multiple_files(paths)`
**Use Cases**:
- Read multiple files simultaneously
- Use when reading a large number of files (e.g., multiple paper html pages)


**Example**:
```bash
python run_fs_ops.py -c "await fs.read_multiple_files(['./a.txt', './b.txt'])"
```

---

### File Writing Tools

#### `write_file(path, content)`
**Use Cases**:
- Create new files
- Overwrite existing files

**⚠️ Warning**: Do NOT include triple backticks (` ``` `) in the content, as this will break command parsing.
**Example**:
```bash
python run_fs_ops.py -c "await fs.write_file('./new.txt', 'Hello World')"
```

---

#### `edit_file(path, edits)`
**Use Cases**:
- Make line-based edits to existing files

**Example**:
```bash
python run_fs_ops.py -c "await fs.edit_file('./file.txt', [{'oldText': 'foo', 'newText': 'bar'}])"
```

---

### Directory Tools

#### `create_directory(path)`
**Use Cases**:
- Create new directories (supports recursive creation)

**Example**:
```bash
python run_fs_ops.py -c "await fs.create_directory('./new/nested/dir')"
```

---

#### `list_directory(path)`
**Use Cases**:
- List all files and directories in a path

**Example**:
```bash
python run_fs_ops.py -c "await fs.list_directory('.')"
```

---

#### `list_files(path=None, exclude_hidden=True)`
**Use Cases**:
- List only files in a directory

**Example**:
```bash
python run_fs_ops.py -c "await fs.list_files('./data')"
```

---

### File Operations

#### `move_file(source, destination)`
**Use Cases**:
- Move or rename files/directories

**Example**:
```bash
python run_fs_ops.py -c "await fs.move_file('./old.txt', './new.txt')"
```

---

#### `search_files(pattern, base_path=None)`
**Use Cases**:
- Search for files matching a glob pattern

**Example**:
```bash
python run_fs_ops.py -c "await fs.search_files('*.txt')"
```

---

### File Information

#### `get_file_info(path)`
**Use Cases**:
- Get detailed metadata (size, created, modified, etc.)

**Example**:
```bash
python run_fs_ops.py -c "await fs.get_file_info('./file.txt')"
```

---

#### `get_file_size(path)`
**Use Cases**:
- Get file size in bytes

**Example**:
```bash
python run_fs_ops.py -c "await fs.get_file_size('./file.txt')"
```

---

#### `get_file_ctime(path)` / `get_file_mtime(path)`
**Use Cases**:
- Get file creation/modification time

**Example**:
```bash
python run_fs_ops.py -c "await fs.get_file_mtime('./file.txt')"
```

---

#### `get_files_info_batch(filenames, base_path=None)`
**Use Cases**:
- Get file information for multiple files in parallel

**Example**:
```bash
python run_fs_ops.py -c "await fs.get_files_info_batch(['a.txt', 'b.txt'], './data')"
```
