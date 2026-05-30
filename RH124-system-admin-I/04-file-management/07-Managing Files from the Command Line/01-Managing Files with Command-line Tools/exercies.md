# Guided Exercise: Manage Files with Command Line Tools — Notes


## Task 1: Create Directories

### Goal: Create `Music`, `Pictures`, and `Videos` directories

### Best practice — `mkdir -pv`:

```bash
mkdir -pv Music Pictures Videos
```

| Option | Meaning |
|--------|---------|
| `-p` | Create parent paths, no error if exists |
| `-v` | Verbose — show what was created |

### Why `-pv` is great:

```bash
mkdir -pv Music Pictures Videos   # Shows: mkdir: created directory 'Music', etc.
mkdir -pv Music Pictures Videos   # Shows: nothing (no error, no output)
```

---

## Task 2: Create Files Using Brace Expansion

### Goal: Create 6 `.mp3`, 6 `.jpg`, and 6 `.avi` files

### ❌ The slow way (don't do this):

```bash
touch song1.mp3 song2.mp3 song3.mp3 song4.mp3 song5.mp3 song6.mp3
```

### ✅ The smart way — Brace expansion `{1..6}`:

```bash
touch song{1..6}.mp3
touch snap{1..6}.jpg
touch film{1..6}.avi
```

### Verify:

```bash
ls -l
```

**Expected files:**
```
song1.mp3  song2.mp3  song3.mp3  song4.mp3  song5.mp3  song6.mp3
snap1.jpg  snap2.jpg  snap3.jpg  snap4.jpg  snap5.jpg  snap6.jpg
film1.avi  film2.avi  film3.avi  film4.avi  film5.avi  film6.avi
```

---

## Task 3: Move Files into Directories

### Goal: Move files to their respective directories

| File Type | Destination |
|-----------|-------------|
| `song*.mp3` | `Music/` |
| `snap*.jpg` | `Pictures/` |
| `film*.avi` | `Videos/` |

### ❌ The tedious way (from student guide):

```bash
mv song1.mp3 song2.mp3 song3.mp3 song4.mp3 song5.mp3 song6.mp3 Music/
```

### ✅ The smart way — Wildcards (`*`):

```bash
mv song*.mp3 Music/
mv snap*.jpg Pictures/
mv film*.avi Videos/
```

### Verify:

```bash
ls -l Music/ Pictures/ Videos/
```

> 💡 `*` = wildcard that matches **any text**. Much faster and less error-prone.

---

## Task 4: Create More Directories

### Goal: Create `friends`, `family`, and `work` directories

```bash
mkdir -pv friends family work
```

---

## 5: Copy Files with Brackets (`[]`)

### Goal: Copy files ending in `1` and `2` into each directory

### The pattern: `[12]` matches either `1` or `2`

### Step 5.1 — Copy to `friends` directory:

```bash
cd friends
cp ~/Music/song[12].mp3 .
cp ~/Pictures/snap[12].jpg .
cp ~/Videos/film[12].avi .
```

| Symbol | Meaning |
|--------|---------|
| `~` | Home directory of current user |
| `[12]` | Match character 1 OR 2 |
| `.` | Current directory (destination) |

### Verify:

```bash
ls -l
```

**Expected:** `song1.mp3`, `song2.mp3`, `snap1.jpg`, `snap2.jpg`, `film1.avi`, `film2.avi`

---

### Step 5.2 — Copy to `family` directory (files 3 & 4):

```bash
cd ../family
cp ~/Music/song[34].mp3 .
cp ~/Pictures/snap[34].jpg .
cp ~/Videos/film[34].avi .
```

> 💡 **Pro tip:** Use `Ctrl+R` to search command history and reuse the previous `cp` command, then edit `[12]` to `[34]`.

### Verify:

```bash
ls -l
```

**Expected:** `song3.mp3`, `song4.mp3`, `snap3.jpg`, `snap4.jpg`, `film3.avi`, `film4.avi`

---

### Step 5.3 — Copy to `work` directory (files 5 & 6):

```bash
cd ../work
cp ~/Music/song[56].mp3 .
cp ~/Pictures/snap[56].jpg .
cp ~/Videos/film[56].avi .
```

### Verify:

```bash
ls -l
```

**Expected:** `song5.mp3`, `song6.mp3`, `snap5.jpg`, `snap6.jpg`, `film5.avi`, `film6.avi`

---

## Task 6: Copy Directories with `cp -r`

### Goal: Copy `friends` and `family` directories into `work`

### Current location: inside `~/work`

```bash
cp -r ~/friends ~/family .
```

| Option | Meaning |
|--------|---------|
| `-r` | Recursive (copy directories + all contents) |

### Verify:

```bash
ls -l
```

**Expected:**
```
family  friends  film5.avi  film6.avi  snap5.jpg  snap6.jpg  song5.mp3  song6.mp3
```

---

## Task 7: View Directory Structure with `tree`

### Go to home directory and display tree:

```bash
cd ~
tree -F
```

### Why `-F`?

| Without `-F` | With `-F` |
|--------------|-----------|
| `Music` | `Music/` |
| `Pictures` | `Pictures/` |
| `Videos` | `Videos/` |

> `-F` adds trailing `/` to directory names — visually clear.

### Expected output:

```
.
├── Music/
│   ├── song1.mp3
│   ├── song2.mp3
│   ...
├── Pictures/
│   ├── snap1.jpg
│   ...
├── Videos/
│   ├── film1.avi
│   ...
├── family/
│   ├── film3.avi
│   ├── snap3.jpg
│   ├── song3.mp3
│   └── film4.avi ...
├── friends/
│   ├── film1.avi
│   ...
└── work/
    ├── family/
    ├── friends/
    ├── film5.avi
    ...
```

---

## Task 8: Clean Up

### Delete the directories (recursive, force):

```bash
rm -rf family friends work
```

| Option | Meaning | Caution |
|--------|---------|---------|
| `-r` | Recursive | ⚠️ |
| `-f` | Force (no prompts) | ⚠️⚠️⚠️ |

### Verify deletion:

```bash
ls -l
```

**Expected:** Only `Music`, `Pictures`, `Videos` remain.

---

# Complete Command Reference Table

| Task | Command(s) |
|------|------------|
| Create directories | `mkdir -pv Music Pictures Videos` |
| Create multiple files | `touch song{1..6}.mp3` |
| Move files (wildcard) | `mv song*.mp3 Music/` |
| Copy with bracket matching | `cp ~/Music/song[12].mp3 .` |
| Copy directories | `cp -r ~/friends ~/family .` |
| View tree structure | `tree -F` |
| Delete directories | `rm -rf family friends work` |

---

# Wildcard & Pattern Cheat Sheet

| Pattern | Matches | Example |
|---------|---------|---------|
| `*` | Any text (including none) | `song*.mp3` → `song1.mp3`, `songXYZ.mp3` |
| `?` | Exactly one character | `song?.mp3` → `song1.mp3` (not `song10.mp3`) |
| `[12]` | Either 1 or 2 | `song[12].mp3` → `song1.mp3`, `song2.mp3` |
| `[34]` | Either 3 or 4 | `song[34].mp3` → `song3.mp3`, `song4.mp3` |
| `{1..6}` | Brace expansion (1,2,3,4,5,6) | `song{1..6}.mp3` (creates 6 files) |

---

# Productivity Tips from This Lab

| Tip | How To |
|-----|--------|
| Create many files | Brace expansion: `{1..100}` |
| Move groups of files | Wildcards: `*.mp3` |
| Copy specific numbered files | Brackets: `[12]` |
| Reuse previous command | `Ctrl+R` (reverse search) |
| Copy directories | `cp -r` |
| See directory structure | `tree -F` |
| Work smarter, not harder | Use patterns, not repetition |

---

# Key Takeaways

- **Brace expansion** (`{1..6}`) = create multiple files in one command.
- **Wildcards** (`*`) = operate on groups of files.
- **Brackets** (`[12]`) = match specific characters (great for numbered files).
- **`cp -r`** = required for copying directories.
- **`tree -F`** = visualize directory structure with trailing `/` indicators.
- **`rm -rf` is powerful but dangerous** — always double-check.
- Work **smarter, not harder** — if you're typing the same thing repeatedly, there's probably a better way.