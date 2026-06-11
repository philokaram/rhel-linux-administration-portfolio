# Editing Files with Vim — Notes

## 1. Why Vim?

| Benefit | Description |
|---------|-------------|
| Universal | Included on **every** Linux installation |
| Fast & lightweight | Runs anywhere, even on minimal servers |
| Always available | No need to install extra software |
| Powerful | Once you learn basics, you'll never be stuck |

> 🎭 **Scary stories?** Yes. But once you know the basics, it's actually really powerful.

---

## 2. Vim Packages

| Package | Provides |
|---------|----------|
| `vim-minimal` | `vi` (Vim's predecessor, less feature-rich) |
| `vim-enhanced` | Full Vim experience (recommended) |

### Check installed packages:

```bash
rpm -qa | grep vim
```

---

## 3. Vim Modes of Operation

| Mode | Purpose | Visual Cue |
|------|---------|------------|
| **Normal** (a.k.a. Command mode) | Navigate, run commands | Blank at bottom left |
| **INSERT** | Type text | `-- INSERT --` |
| **Replace** | Overwrite text | `-- REPLACE --` |
| **Command-line** (Extended) | Save, quit, substitute | `:` prompt |
| **Visual** | Manipulate text at scale | `-- VISUAL --` or `-- VISUAL BLOCK --` |

> 💡 **Default mode = Normal mode** (when you first open Vim)

---

## 4. Opening a File

```bash
vim filename.txt
```

- If file exists → opens it
- If file doesn't exist → creates new file (when saved)

---

## 5. Basic Workflow — Edit, Save, Quit

### Step 1: Open file

```bash
vim filename.txt
```

### Step 2: Enter INSERT mode

Press `i` (insert at cursor)

### Step 3: Type your text

```
Red Hat
```

### Step 4: Return to Normal mode

Press `Esc`

### Step 5: Save and quit

Press `:` then type `wq`

```
:wq
```

| Command | Meaning |
|---------|---------|
| `:w` | **W**rite (save) |
| `:q` | **Q**uit |
| `:wq` | Write + Quit |
| `:q!` | Quit without saving (force) |

---

## 6. Entering INSERT Mode — Multiple Ways

| Key | Action |
|-----|--------|
| `i` | Insert **at** cursor |
| `I` | Insert at **beginning of line** |
| `a` | Insert **after** cursor |
| `A` | Insert at **end of line** |
| `o` | Insert **below** current line (opens new line) |
| `O` | Insert **above** current line (opens new line) |

> 💡 `O` is great for adding a line above where you are.

---

## 7. Normal Mode Commands (Navigation & Actions)

| Command | Action |
|---------|--------|
| `yy` | **Y**ank (copy) current line |
| `p` | **P**aste below cursor |
| `P` | Paste above cursor |
| `dd` | **D**elete current line |
| `dw` | Delete **w**ord |
| `u` | **U**ndo |
| `Ctrl+R` | **R**edo |
| `gg` | Go to **top** of file |
| `G` | Go to **bottom** of file |
| `:number` | Go to specific line number (e.g., `:42`) |
| `/keyword` | Search forward for keyword |
| `n` | Next search match |
| `N` | Previous search match |

### Copy/paste example:

```vim
yy          " Copy (yank) current line
10p         " Paste 10 times
u           " Undo
```

---

## 8. Visual Mode — Manipulating Text at Scale

### Visual Line Mode (`Shift+V`)

```vim
Shift+V     " Enter VISUAL LINE mode
(arrow keys) " Highlight multiple lines
d           " Delete highlighted lines
```

### Visual Block Mode (`Ctrl+V`)

```vim
Ctrl+V      " Enter VISUAL BLOCK mode
(arrows)    " Highlight a rectangular block
d           " Delete block
```

> 💡 Great for deleting columns of text (like comments at start of lines).

### Repeat last command — `.` (period/full stop)

After deleting a word with `dw`, press `.` to delete the next word.

---

## 9. Command-Line Mode — Advanced Operations

### Execute shell command and insert output:

```vim
:r! command
```

**Example — insert first 30 lines of /etc/services:**

```vim
:r! head -30 /etc/services
```

| Part | Meaning |
|------|---------|
| `:r!` | **R**ead output of external command |
| `head -30 /etc/services` | Command to run |

### Search and replace:

```vim
:%s/old/new/g
```

| Part | Meaning |
|------|---------|
| `%` | Entire file |
| `s` | **S**ubstitute |
| `/old/` | What to find |
| `/new/` | Replace with |
| `/g` | **G**lobal (all occurrences on line) |

**Example — replace all "sink" with "swim":**

```vim
:%s/sink/swim/g
```

---

## 10. Vim Configuration — `~/.vimrc`

Create a configuration file to customize Vim behavior.

### Example `.vimrc` settings:

```vim
set number          " Show line numbers
set expandtab       " Use spaces instead of tabs
set tabstop=2       " Tab = 2 spaces
set shiftwidth=2    " Indent = 2 spaces
set ignorecase      " Searches case-insensitive
set incsearch       " Highlight search matches as you type
```

### Apply settings:

Save to `~/.vimrc` — settings load automatically each time Vim starts.

---

## 11. VimTutor — Interactive Tutorial

```bash
vimtutor
```

- Interactive tutorial built into Linux
- Recommended time: **30-60 minutes**
- Best way to learn Vim hands-on

> 💡 Run `vimtutor` on any Linux system to practice.

---

# Quick Reference — Most Important Commands

| Task | Command |
|------|---------|
| Open file | `vim filename.txt` |
| Enter INSERT mode | `i` |
| Return to Normal mode | `Esc` |
| Save file | `:w` |
| Quit | `:q` |
| Save & quit | `:wq` |
| Quit without saving | `:q!` |
| Copy line | `yy` |
| Paste below | `p` |
| Delete line | `dd` |
| Undo | `u` |
| Redo | `Ctrl+R` |
| Search | `/keyword` |
| Go to top | `gg` |
| Go to bottom | `G` |
| Repeat last command | `.` |

---

# Mode Transition Diagram

```
        ┌─────────────────────────────────────────┐
        │                                         │
        ▼                                         │
   ┌─────────┐      i,I,a,A,o,O      ┌──────────┐ │
   │ NORMAL  │ ─────────────────────► │  INSERT  │ │
   │  MODE   │ ◄───────────────────── │   MODE   │ │
   └─────────┘          Esc           └──────────┘ │
        │                                         │
        │ :                                       │
        ▼                                         │
   ┌─────────────┐                               │
   │ COMMAND-LINE│                               │
   │    MODE     │                               │
   └─────────────┘                               │
        │                                         │
        │ (after command, returns to Normal)     │
        └─────────────────────────────────────────┘
```

---

# Key Takeaways

- **Vim is universal** — installed on every Linux system. Learn it once, use it everywhere.
- **Normal mode is default** — you start here. Press `Esc` to return.
- **INSERT mode** = typing text. Press `i` to enter.
- **Command-line mode** = save, quit, search/replace. Press `:` to enter.
- **Visual mode** = manipulate blocks of text. `Shift+V` (lines) or `Ctrl+V` (blocks).
- **`yy`** = copy line, **`p`** = paste, **`dd`** = delete line, **`u`** = undo.
- **`/keyword`** = search, **`n`** = next match.
- **`:r! command`** = insert output of a shell command into your file.
- **`:%s/old/new/g`** = search and replace across entire file.
- **`~/.vimrc`** = personal configuration file (line numbers, tab settings, etc.).
- **`vimtutor`** = 30-60 minute interactive tutorial — best way to learn.

> 💡 Master the basics (open, edit, save, quit) and you'll be comfortable editing files anywhere on a Linux system. The advanced features will come with practice.