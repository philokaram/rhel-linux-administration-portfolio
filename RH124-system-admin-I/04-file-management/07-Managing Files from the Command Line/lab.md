# End of Chapter Lab: Manage Files from the Command Line — Solution Notes


## Task 1: Create Project Directories

### Create `~/Documents/project_plans`:

```bash
mkdir -pv ~/Documents/project_plans
```

| Option | Meaning |
|--------|---------|
| `-p` | Create parent directories if needed |
| `-v` | Verbose (show what was created) |

---

## Task 2: Create Project Plan Files

### Command:

```bash
touch ~/Documents/project_plans/season{1..2}_project_plan.odf
```

### Brace expansion `{1..2}` creates:

- `season1_project_plan.odf`
- `season2_project_plan.odf`

---

## Task 3: Create TV Episode Files

### Command:

```bash
touch ~/tv_season{1..2}_episode{1..6}.ogg
```

### What this creates:

| Season | Episodes |
|--------|----------|
| Season 1 | `tv_season1_episode1.ogg` ... `tv_season1_episode6.ogg` |
| Season 2 | `tv_season2_episode1.ogg` ... `tv_season2_episode6.ogg` |

**Total:** 12 files

---

## Task 4: Create Mystery Chapter Files

### Command:

```bash
touch ~/mystery_chapter{1..8}.odf
```

### Creates:

- `mystery_chapter1.odf` through `mystery_chapter8.odf`

---

## Task 5: Create Video Season Directories

### Command:

```bash
mkdir -pv ~/Videos/season{1,2}
```

### Creates:

```
~/Videos/
├── season1/
└── season2/
```

---

## Task 6: Move Episode Files to Season Directories

### Move season 1 episodes:

```bash
mv ~/tv_season1_episode*.ogg ~/Videos/season1/
```

### Move season 2 episodes:

```bash
mv ~/tv_season2_episode*.ogg ~/Videos/season2/
```

> 💡 Use `up arrow` then edit `1` to `2` — work smarter!

---

## Task 7: Create Bestseller Directory Structure

### Command:

```bash
mkdir -pv ~/Documents/my_bestseller/{chapters,editor,changes,vacation}
```

### Result:

```
~/Documents/my_bestseller/
├── chapters/
├── editor/
├── changes/
└── vacation/
```

---

## Task 8: Move Mystery Chapters into Chapters Directory

### Change to chapters directory:

```bash
cd ~/Documents/my_bestseller/chapters/
```

### Move all mystery chapters:

```bash
mv ~/mystery_chapter*.odf .
```

| Symbol | Meaning |
|--------|---------|
| `~` | Home directory |
| `*` | Wildcard (all mystery_chapter files) |
| `.` | Current directory |

---

## Task 9: Move First Two Chapters to Editor Directory

### Command:

```bash
mv mystery_chapter{1,2}.odf ../editor/
```

| Symbol | Meaning |
|--------|---------|
| `{1,2}` | Brace expansion (file1 and file2) |
| `..` | Parent directory |
| `/editor/` | Destination |

---

## Task 10: Move Last Two Chapters to Vacation Directory

### Command:

```bash
mv mystery_chapter{7,8}.odf ../vacation/
```

---

## Task 11: Copy Episode to Vacation Directory

### Change to season2 directory:

```bash
cd ~/Videos/season2/
```

### Copy episode1:

```bash
cp tv_season2_episode1.ogg ~/Documents/my_bestseller/vacation/
```

### Verify and return:

```bash
ls ~/Documents/my_bestseller/vacation/
cd -        # Returns to previous directory (~/Videos/season2/)
```

> 💡 `cd -` = go back to previous working directory (not tilde!)

---

## Task 12: Copy Chapters 5 & 6 to Changes Directory

### Command:

```bash
cp ~/Documents/my_bestseller/chapters/mystery_chapter[56].odf ~/Documents/my_bestseller/changes/
```

| Pattern | Matches |
|---------|---------|
| `[56]` | Either `5` OR `6` |

### Change to changes directory:

```bash
cd ~/Documents/my_bestseller/changes/
```

---

## Task 13: Add Date Extension to Files (Brace + Command Substitution)

### Copy chapter5 with date stamp (`+%F` = YYYY-MM-DD):

```bash
cp mystery_chapter5.odf mystery_chapter5_$(date +%F).odf
```

### Copy chapter5 with epoch seconds (`+%s`):

```bash
cp mystery_chapter5.odf mystery_chapter5_$(date +%s).odf
```

| `date` format | Output Example | Meaning |
|---------------|----------------|---------|
| `+%F` | `2025-01-15` | YYYY-MM-DD |
| `+%s` | `1736956800` | Seconds since 1970-01-01 |

### Command substitution syntax:

```bash
$(command)      # Run command and insert its output
```

### Alternative brace expansion trick (shown in video):

```bash
cp mystery_chapter5.odf mystery_chapter5_{$(date +%F)}.odf
```

---

## Task 14: Delete Changes Directory Contents

### Go to home directory:

```bash
cd ~
```

### Delete all files in changes directory:

```bash
rm ~/Documents/my_bestseller/changes/*
```

### Delete the empty changes directory:

```bash
rmdir ~/Documents/my_bestseller/changes/
```

> ⚠️ `rmdir` only works on **empty** directories.

### Alternative (if not empty — but ours is empty):

```bash
rm -r ~/Documents/my_bestseller/changes/
```

---

## Task 15: Delete Vacation Directory

```bash
rm -r ~/Documents/my_bestseller/vacation/
```

| Option | Meaning |
|--------|---------|
| `-r` | Recursive (delete directory + contents) |

---

## Task 16: Create Backups Directory

```bash
mkdir -pv ~/Documents/backups
```

---

## Task 17: Create Hard Link to Project Plan

### Command:

```bash
ln ~/Documents/project_plans/season2_project_plan.odf ~/Documents/backups/season2_project_plan.odf
```

### Verify with inode check:

```bash
ls -li ~/Documents/project_plans/season2_project_plan.odf ~/Documents/backups/season2_project_plan.odf
```

**Expected:** Same inode number, link count = 2

---


---

# Complete Command Reference Table

| Task | Command(s) |
|------|------------|
| Create directory | `mkdir -pv ~/Documents/project_plans` |
| Create files (brace) | `touch ~/Documents/project_plans/season{1..2}_project_plan.odf` |
| Create episode files | `touch ~/tv_season{1..2}_episode{1..6}.ogg` |
| Create chapter files | `touch ~/mystery_chapter{1..8}.odf` |
| Create video dirs | `mkdir -pv ~/Videos/season{1,2}` |
| Move with wildcard | `mv ~/tv_season1_episode*.ogg ~/Videos/season1/` |
| Create nested dirs | `mkdir -pv ~/Documents/my_bestseller/{chapters,editor,changes,vacation}` |
| Move with brace | `mv mystery_chapter{1,2}.odf ../editor/` |
| Copy with bracket | `cp *[56].odf ~/Documents/my_bestseller/changes/` |
| Date stamp copy | `cp file.odf file_$(date +%F).odf` |
| Delete directory | `rm -r ~/Documents/my_bestseller/vacation/` |
| Create hard link | `ln source link` |

---

# Key Expansion Patterns Used in This Lab

| Pattern | Purpose | Example |
|---------|---------|---------|
| `{1..2}` | Brace expansion (range) | `season{1..2}` |
| `{1,2}` | Brace expansion (list) | `{1,2}` |
| `*` | Wildcard (any text) | `tv_season1_episode*.ogg` |
| `[56]` | Bracket (character set) | `*[56].odf` |
| `$(command)` | Command substitution | `$(date +%F)` |
| `..` | Parent directory | `../editor/` |
| `-` | Previous directory | `cd -` |
| `~` | Home directory | `~/Documents/` |
| `.` | Current directory | `mv file .` |

---

# Key Takeaways

- **Brace expansion** (`{1..2}`) and **wildcards** (`*`) save massive typing.
- **Brackets** (`[56]`) match specific characters — great for numbered files.
- **Command substitution** (`$(date +%F)`) adds dynamic content to filenames.
- **`cd -`** returns to previous directory (not home).
- **Hard links** share the same inode — verify with `ls -li`.
- **`rmdir`** only deletes empty directories; use `rm -r` for non-empty.
