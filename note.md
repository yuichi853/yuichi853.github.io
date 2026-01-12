# Downloads and initializes Git submodules

```bash
git submodule update --init --recursive
```

This command **downloads and initializes Git submodules** that are registered in the repository
(e.g. the **PaperMod** theme used by a Hugo site).

---

## Why this is necessary

* The Hugo theme (PaperMod) is managed as a **Git submodule**
* Running `git clone` **does NOT automatically download submodule contents**
* As a result:

  * `themes/PaperMod` exists but is empty
  * Hugo cannot find the theme
  * Pages, layouts, or images may not render correctly

---

## How to detect the problem

Run:

```bash
git submodule status
```

Example output:

```text
-7d061d56d4664bd9c8241eb904994c98b928f0c8 themes/PaperMod
```

Meaning of the prefix:

* `-` (dash) → **submodule is registered but NOT initialized**
* no prefix / space → submodule is properly checked out
