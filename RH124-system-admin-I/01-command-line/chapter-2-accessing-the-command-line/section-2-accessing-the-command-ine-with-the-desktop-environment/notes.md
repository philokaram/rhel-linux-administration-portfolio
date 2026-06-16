# GNOME Desktop in RHEL — Notes

## 1. What is GNOME?

* GNOME is the **graphical user interface (GUI)** in Red Hat Enterprise Linux.
* It is **optional** (servers can run headless without it).
* Provides a desktop environment for easier interaction.

---

## 2. Main Menu (Red Hat Logo)

* Located at the **top-left corner**.
* Similar to the **Start menu in Windows**.
* Opens a dock with favorite applications.

---

## 3. Dock (Favorites Bar)

* Appears at the bottom after opening the menu.
* Contains frequently used apps.
* You can:

  * Add apps (drag & drop)
  * Rearrange icons

### Common apps:

* Terminal (Ptyxis in RHEL 10)
* Firefox browser
* Text editor
* Calculator
* Visual Studio Code
* App grid (shows all installed applications)

---

## 4. App Grid

* Accessed via **3×3 grid icon**
* Shows all installed applications

---

## 5. Search Feature (“Type to Search”)

* Found in the main menu
* Can search:

  * Applications (e.g., Firefox)
  * Settings (e.g., “Displays”)
* Quick way to access system tools

---

## 6. Settings Menu (Top Right Corner)

* Used to configure system behavior:

  * Display settings (resolution, scaling)
  * Network settings (wired/wireless)
  * Power settings
  * Appearance (dark/light mode)

---

## 7. Screenshot Tool

* Located in top-right system menu
* Can capture:

  * Full screen
  * Selected area
  * Window
* Can also do screen recording (depending on system)

---

## 8. Screen Lock

* Locks the system when away
* Requires password to unlock

---

## 9. Network Settings

* Manage wired or wireless connections
* View connection status
* Configure network settings

---

## 10. Power & Performance Settings

* Power modes:

  * **Balanced**
  * **Power Saver** (reduces performance, saves battery)
* Used especially on laptops

---

## 11. System Power Options

From top-right menu:

* Power Off
* Restart
* Suspend
* Log Out

---

## 12. Notifications (Message Tray)

* Located at the **top center (date/time area)**
* Shows:

  * System messages
  * Notifications (mail, chat, etc.)
* Includes **Do Not Disturb mode**

---

## 13. Workspaces (Virtual Desktops)

* Allow multiple “screens” on one monitor
* Useful for organizing tasks

### Example:

* Workspace 1 → Terminal
* Workspace 2 → Firefox
* Workspace 3 → VS Code

### Key features:

* Drag apps between workspaces
* New workspace is created automatically when needed

### Keyboard shortcuts:

* `Ctrl + Alt + Left Arrow` → switch workspace left
* `Ctrl + Alt + Right Arrow` → switch workspace right

---

## 14. Virtual Consoles (TTY)

Linux provides multiple terminals:

| TTY       | Purpose                |
| --------- | ---------------------- |
| tty1      | Graphical login screen |
| tty2      | Graphical session      |
| tty3–tty6 | Text-based logins      |

### Switching:

* `Ctrl + Alt + F1–F6`

---

## 15. Text-Based Login (TTY)

* You can log in without GUI
* Example:

  * Username: `student`
  * Password: `student`
* Always remember to log out when done

---

## 16. Logout Shortcut

* `Ctrl + D` → exits session quickly
* Equivalent to `exit`

---

## 17. GUI Login vs Headless Systems

* GUI login = graphical desktop environment
* Headless = no GUI (server-only systems)
* Many servers run headless for efficiency and performance

---

## 18. Security Note

* Auto-login is convenient but **not secure**
* Manual login is preferred in real systems

---

## 19. Key Takeaways

* GNOME provides a full desktop environment in Linux
* Workspaces help organize multiple tasks
* Top-right menu controls system settings and power
* TTYs allow switching between graphical and text environments
* Keyboard shortcuts improve efficiency

---
