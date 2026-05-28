# Linux Man Pages & Documentation — Notes

## 1. What Are Man Pages?

* **Man pages** (“manual pages”) are built-in Linux documentation.
* They explain:

  * Commands
  * Options
  * Arguments
  * Configuration files
  * System administration tools

👉 You do **not** need to memorize every command.

---

## 2. Man Page Database (`mandb`)

Sometimes man pages are unavailable because the database is not initialized.

### Initialize/update database:

```bash
sudo mandb
```

### What it does:

* Scans documentation directories:

```bash
/usr/share/man/
```

* Indexes all installed man pages

---

## 3. Searching for Man Pages

Use:

```bash
man -k keyword
```

Example:

```bash
man -k foo
```

### Purpose:

* Searches man page names and descriptions
* Similar to a keyword search

---

## 4. Opening a Man Page

Basic syntax:

```bash
man command
```

Example:

```bash
man tar
```

---

## 5. Navigating Inside Man Pages

Man pages use the **less** pager.

### Useful keys:

| Key        | Action         |
| ---------- | -------------- |
| `/keyword` | Search         |
| `n`        | Next match     |
| `N`        | Previous match |
| `PageDown` | Next page      |
| `PageUp`   | Previous page  |
| `gg`       | Go to top      |
| `G`        | Go to bottom   |
| `Home`     | Beginning      |
| `End`      | End            |
| `q`        | Quit           |

---

## 6. Man Page Sections

Different types of documentation are grouped into sections.

### Important sections:

| Section | Meaning                        |
| ------- | ------------------------------ |
| 1       | User commands / shell commands |
| 5       | Configuration files & formats  |
| 8       | System administration commands |

---

## 7. Viewing Config File Documentation

Example:

```bash
man 5 crontab
```

### Meaning:

* `5` → config file section
* `crontab` → cron configuration documentation

---

## 8. Learn About the `man` Command Itself

Yes, Linux lets you read the manual for the manual:

```bash
man man
```

This explains:

* Syntax
* Sections
* Navigation
* Usage

---

## 9. Important Ideas

* Documentation is already installed locally
* Use man pages before searching online
* Great for troubleshooting and exams
* Especially useful for:

  * remembering options
  * config syntax
  * admin commands

---

# Important Commands Summary

| Command          | Purpose                   |
| ---------------- | ------------------------- |
| `man tar`        | Open tar manual           |
| `man -k keyword` | Search man pages          |
| `man 5 crontab`  | Open config file manual   |
| `man man`        | Manual for man            |
| `sudo mandb`     | Build/update man database |

---

# Key Takeaways

* Linux has excellent built-in documentation
* `man` is one of the most important Linux commands
* Learn navigation shortcuts to save time
* Use sections (`1`, `5`, `8`) to find the right documentation
* “Work smarter, not harder” → use the docs instead of memorizing everything
