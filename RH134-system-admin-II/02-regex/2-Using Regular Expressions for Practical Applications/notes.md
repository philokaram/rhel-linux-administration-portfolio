# Matching Text with Regular Expressions — Notes

## 1. What Is `grep`?

- **grep** = **G**lobal **R**egular **E**xpression **P**rint
- A tool for finding patterns in text
- `grep` is aliased to `grep --color=auto` (highlights matches)

### Check the alias:

```bash
type grep
```

**Output:** `grep is aliased to 'grep --color=auto'`

---

## 2. Basic grep — Simple Patterns

### Find a string anywhere in a line:

```bash
grep learn /usr/share/dict/words
```

### Return codes:

| Code | Meaning |
|------|---------|
| `0` | Pattern found |
| `1` | Pattern not found |
| `2` | File not found |

---

## 3. Anchors — `^` (Beginning) and `$` (End)

| Anchor | Meaning |
|--------|---------|
| `^` | Beginning of line |
| `$` | End of line |

### Examples:

```bash
grep ^learn /usr/share/dict/words    # Lines starting with "learn"
grep learn$ /usr/share/dict/words    # Lines ending with "learn"
grep ^learn$ /usr/share/dict/words   # Lines that are exactly "learn"
```

---

## 4. Character Classes — `[ ]` (Brackets)

### Match any one character from a set:

```bash
grep ^learn[ed] /usr/share/dict/words   # learn + e OR d
```

**Matches:** `learned`, `learner`

### Negation inside brackets (`^`):

```bash
grep ^learn[^e] /usr/share/dict/words   # learn + NOT e
```

---

## 5. Word Boundaries — `\<` and `\>`

| Boundary | Meaning |
|----------|---------|
| `\<` | Beginning of a word |
| `\>` | End of a word |

### Example — match whole word "root":

```bash
grep \<root\> /etc/passwd
```

**Matches:** `root` as a whole word (not `groot`)

### Combine with anchor:

```bash
grep ^\<root\> /etc/passwd
```

**Matches:** Lines beginning with the whole word `root`

---

## 6. Wildcards — `.` (Any Single Character)

| Symbol | Meaning |
|--------|---------|
| `.` | Any single character (wildcard) |

### Example:

```bash
grep ^learn. /usr/share/dict/words   # "learn" + any single character
```

**Matches:** `learned`, `learner`, etc.

---

## 7. Modifiers — `*` (Zero or More)

| Symbol | Meaning |
|--------|---------|
| `*` | Zero or more of the previous character |

### Example:

```bash
grep ^learne.* /usr/share/dict/words
```

**Matches:** `learned`, `learner`, `learning`, etc.

> 💡 `.` = any character, `*` = 0 or more → `.*` = any number of any characters.

---

## 8. Extended Regular Expressions — `grep -E` or `egrep`

### Extended features:

| Feature | Symbol | Meaning |
|---------|--------|---------|
| One or more | `+` | One or more of previous character |
| Optional | `?` | Zero or one of previous character |
| Alternation | `\|` | OR |
| Grouping | `()` | Group expressions |

### Example — `+` (one or more):

```bash
grep -E ^learne.+ /usr/share/dict/words
```

### Without `-E`, escape the `+`:

```bash
grep ^learne.\+ /usr/share/dict/words
```

> 💡 `grep -E` and `egrep` are equivalent.

---

## 9. Character Classes — `[[:class:]]`

| Class | Meaning | Example |
|-------|---------|---------|
| `[[:alnum:]]` | Alphanumeric | `a-z, A-Z, 0-9` |
| `[[:alpha:]]` | Alphabetical | `a-z, A-Z` |
| `[[:digit:]]` | Digits | `0-9` |
| `[[:space:]]` | Whitespace | Space, tab, newline |
| `[[:lower:]]` | Lowercase | `a-z` |
| `[[:upper:]]` | Uppercase | `A-Z` |

### Example — line starts with exactly 2 letters followed by 3 digits:

```bash
grep -E ^[[:alpha:]]{2}[[:digit:]]{3}$ /usr/share/dict/words
```

---

## 10. Quantifiers — `{n}` (Exact Count)

| Quantifier | Meaning |
|------------|---------|
| `{n}` | Exactly n times |
| `{n,}` | n or more times |
| `{,m}` | m or fewer times |
| `{n,m}` | Between n and m times |

### Example:

```bash
grep -E ^[[:alpha:]]{2}[[:digit:]]{3} /usr/share/dict/words
```

**Matches:** Lines starting with exactly 2 letters followed by exactly 3 digits.

---

## 11. Alternation — `|` (OR)

### Match word1 OR word2:

```bash
grep -E '\<(learn|certify)\>' /usr/share/dict/words
```

**Matches:** `learn`, `certify` as whole words.

### With anchors:

```bash
grep -E '^\<(learn|certify)\>' /usr/share/dict/words
```

**Matches:** Lines beginning with `learn` or `certify`.

---

## 12. Negation — `-v` (Invert Match)

### Show lines that do NOT match:

```bash
grep -v nologin /etc/passwd
```

### Negation with extended regex:

```bash
grep -vE 'nologin|sync|shutdown|halt' /etc/passwd
```

---

## 13. Decommenting Configuration Files

### The problem:

- Configuration files have comments (`#`, `;`, `//`)
- You want to see only **active** configuration

### Simple decomment (hash only):

```bash
grep -v '^#' /etc/ssh/sshd_config | grep -v '^$'
```

> ⚠️ This fails for comments with leading spaces or other comment characters.

### Advanced decomment alias:

```bash
alias decomment='egrep -v "^(#|$|;|//)"'
```

### Use:

```bash
decomment /etc/ssh/sshd_config
```

---

## 14. Extended Decomment (with spaces/handling)

### For configuration files with spaces before comments:

```bash
egrep -v "^(|[[:space:]]*)(#|;|//)" /etc/ssh/sshd_config
```

| Part | Meaning |
|------|---------|
| `^` | Beginning of line |
| `[[:space:]]*` | Zero or more spaces/tabs |
| `(#|;|//)` | Comment character(s) |
| `.*` | Rest of the line |
| `?` | Make the group optional |
| `$` | End of line |

> 💡 This pattern handles: `#comment`, `  #comment`, `;comment`, `//comment`.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Basic grep | `grep pattern file` |
| Case-insensitive grep | `grep -i pattern file` |
| Invert match | `grep -v pattern file` |
| Extended regex | `grep -E pattern file` or `egrep pattern file` |
| Beginning of line | `grep ^pattern file` |
| End of line | `grep pattern$ file` |
| Whole word | `grep \<pattern\> file` |
| Any single character | `grep . pattern file` |
| Zero or more | `grep * pattern file` |
| One or more | `grep -E + pattern file` |
| Optional | `grep -E ? pattern file` |
| Exact count | `grep -E {n} pattern file` |
| Alternation (OR) | `grep -E '(pattern1|pattern2)' file` |
| Character class | `grep [[:digit:]] file` |
| Decomment (hash only) | `grep -v '^#' file \| grep -v '^$'` |

---

# Regular Expression Symbols — Quick Reference

| Symbol | Meaning | Example |
|--------|---------|---------|
| `^` | Beginning of line | `^root` |
| `$` | End of line | `root$` |
| `.` | Any single character | `le.rn` |
| `*` | Zero or more of previous | `.*` |
| `\+` / `+` | One or more of previous | `.+` |
| `\?` / `?` | Zero or one of previous | `.?` |
| `\<` | Beginning of word | `\<root` |
| `\>` | End of word | `root\>` |
| `[ ]` | Character set | `[aeiou]` |
| `[^ ]` | Negated character set | `[^aeiou]` |
| `[[:class:]]` | Character class | `[[:digit:]]` |
| `{n}` | Exactly n times | `{2}` |
| `\|` / `\|` | OR (alternation) | `learn\|certify` |
| `\( \)` / `( )` | Grouping | `(learn\|certify)` |

---

# Character Classes Reference

| Class | Matches |
|-------|---------|
| `[[:alnum:]]` | Alphanumeric (letters + digits) |
| `[[:alpha:]]` | Alphabetical letters |
| `[[:digit:]]` | Digits (0-9) |
| `[[:space:]]` | Whitespace (space, tab, newline) |
| `[[:lower:]]` | Lowercase letters |
| `[[:upper:]]` | Uppercase letters |
| `[[:punct:]]` | Punctuation characters |
| `[[:print:]]` | Printable characters |

---

# Other Commands That Understand Regular Expressions

| Command | Purpose |
|---------|---------|
| `sed` | Streaming editor — non-interactive file editing |
| `awk` | Powerful text processing language |
| `less` | Pager with regex search (`/pattern`) |
| `vi`/`vim` | Text editor with regex search/replace |
| `grep` | Search for patterns in text |

---

# Key Takeaways

- **`grep`** = Global Regular Expression Print — searches for patterns in text.
- **Anchors** (`^`, `$`) = beginning and end of line.
- **Word boundaries** (`\<`, `\>`) = beginning and end of word.
- **Character classes** (`[ ]`) = match any one character from a set.
- **Wildcard** (`.`) = any single character.
- **Modifiers** (`*`, `+`, `?`, `{n}`) = control how many times to match.
- **Extended regex** (`grep -E` or `egrep`) = supports `+`, `?`, `|`, `()` without escaping.
- **Alternation** (`|`) = OR.
- **Character classes** (`[[:digit:]]`) = pre-defined groups.
- **Negation** (`-v`) = invert match (show lines that do NOT match).
- **Decommenting** = filtering out comments from config files (useful for troubleshooting).
- **Other tools** (`sed`, `awk`) also understand regular expressions.
- **Practice** — regular expressions are best learned by examples and experimentation.