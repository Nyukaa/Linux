## 1. Reading permissions

```
-rw-rw-r--. 1 student student 0 Mar  8 17:36 test
```

| Part              | Meaning                                    |
| ----------------- | ------------------------------------------ |
| `-`               | file type (`-` file, `d` dir, `l` symlink) |
| `rw-`             | user (owner)                               |
| `rw-`             | group                                      |
| `r--`             | other                                      |
| `student student` | owner, group owner                         |

| Perm | On a file      | On a directory        |
| ---- | -------------- | --------------------- |
| `r`  | read content   | list file names       |
| `w`  | change content | create / delete files |
| `x`  | run as program | `cd` into it          |

- Order: user, then group, then other. The first matching set wins.
- To delete a file you need `w` + `x` on the **directory**, not on the file.
- Permissions are not inherited by subdirectories.
- `ls -ld dir` shows permissions of the directory itself.
