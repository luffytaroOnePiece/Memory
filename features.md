# Memory Bank — Features Documentation

A comprehensive list of all features available in the Memory Bank application.

---

## 🌳 File & Folder Tree

- **Hierarchical tree view** — Browse your knowledge repository as an interactive file/folder tree built from `index.json`.
- **Expand/Collapse folders** — Click any folder to toggle its contents open or closed.
- **Expand All / Collapse All** — One-click button (or press **`E`**) to expand or collapse every folder at once.
- **Folder item counts** — Each folder shows a badge with the total number of files it contains (recursively).
- **Sorted entries** — Folders and files are sorted alphabetically for easy scanning.

-----

## 🔍 Search

- **Real-time search** — Type in the search bar to instantly filter files and folders by name.
- **Highlight matches** — Matching text is highlighted in blue within results.
- **Auto-expand parents** — When a match is found, all parent folders are automatically opened.
- **Keyboard focus** — Press **`/`** from anywhere on the page to jump to the search bar.
- **Escape to blur** — Press **`Esc`** to unfocus the search input.

---

## 📄 Document Viewer

- **In-page viewer** — Click any file to open it in a full-screen iframe overlay without leaving the page.
- **Breadcrumb trail** — The viewer's top bar shows the full path: `Memory Bank › Folder › File Title`.
- **Back button** — Click the ← arrow or press **`Esc`** to close the viewer and return to the tree.
- **ESC hint** — A subtle `ESC` badge in the top bar reminds you of the keyboard shortcut.

---

## ⏱ Recently Viewed

- **Recent files** — The last 5 files you opened are shown as quick-access cards above the tree.
- **Persistent history** — Recently viewed items are saved in `localStorage` and persist across sessions.
- **One-click reopen** — Click any recent card to instantly reopen that file in the viewer.

---

## ⭐ Bookmarks

- **Bookmark files** — Hover over any file to reveal a ⭐ star button. Click it to bookmark the file.
- **Bookmarks section** — Pinned files appear in a dedicated "Bookmarks" section at the top of the page.
- **Quick access** — Click a bookmark card to open the file directly in the viewer.
- **Remove bookmarks** — Hover over a bookmark card and click the ✕ to un-bookmark, or click the star again in the tree.
- **Persistent** — Bookmarks are saved in `localStorage` and persist across sessions and page reloads.

---

## 🌗 Dark / Light Theme

- **Theme toggle** — Click the ☀️/🌙 button in the toolbar to switch between dark and light themes.
- **Persistent preference** — Your theme choice is saved in `localStorage` and applied on next visit.
- **Full coverage** — The theme applies to the tree, search, viewer overlay, bookmarks, and all UI elements.

---

## ⌨️ Keyboard Shortcuts

Press **`?`** at any time to open the keyboard shortcuts help overlay.

| Key   | Action                                 |
| ----- | -------------------------------------- |
| `/`   | Focus search input                     |
| `Esc` | Close viewer / shortcuts / blur search |
| `?`   | Toggle keyboard shortcuts help         |
| `E`   | Expand / Collapse all folders          |

---

## ⬆️ Scroll to Top

- **Floating button** — A ↑ button appears in the bottom-right corner after scrolling down 300px.
- **Smooth scroll** — Click it to smoothly scroll back to the top of the page.
- **Auto-hide** — The button disappears when you're already at the top.

---

## 🎆 Visual Design

- **Glassmorphism** — Frosted-glass surfaces with `backdrop-filter: blur()` for a premium feel.
- **Ambient animated background** — Floating gradient blobs create a dynamic, living background.
- **Shine effect** — Hover over files and folders for a subtle horizontal light sweep.
- **Micro-animations** — Smooth transitions on hover, expand/collapse, and scroll interactions.
- **Google Fonts (Outfit)** — Clean, modern typography throughout.
- **Custom scrollbars** — Styled scrollbars that match the overall dark/light theme.
