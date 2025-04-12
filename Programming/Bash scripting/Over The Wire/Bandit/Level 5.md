**Password**: 4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw
**Tools**: [[find]], [[xargs]]

## Solution
The password for the [[Level 6]] is stored in a file somewhere under the **inhere** directory and has all of the following properties
- human-readable
- 1033 bytes in size
- not executable

with [[find]] we can filter files fulfilling with all this conditions and with [[xargs]] we can concatenate [[cat]] to the output of the find command to read immediately the file.

```bash
find ./inhere/ -size 1033c ! -executable -type f | xargs cat
```