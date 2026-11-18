# leet-mode

A LeetCode client that lives inside Emacs. Browse problems, pick a language,
solve them in a real file with a real language server attached, and submit
without leaving the editor.

## Why

The browser editor is fine until you want the things Emacs already gives you:
completion, go-to-definition, your own keybindings, your own snippets. The hard
part is that a LeetCode solution is a class fragment with no imports and no
project around it, which is exactly the situation language servers handle worst.
leet-mode's central design decision is to never present a scratch buffer: every
solution is written into a real file inside a per-language workspace that looks
like an ordinary project, so `eglot` attaches on its own with no special casing.

## Goals

- **Authentication by browser login.** Open the LeetCode login page, let the
  user log in as a human, then scrape `LEETCODE_SESSION` and `csrftoken` from
  the browser's cookie store. Credentials are cached in `~/.authinfo.gpg`.
  A manual paste path always remains available as a fallback.
- **Problem browser.** A `tabulated-list-mode` buffer of problems with id,
  title, difficulty, acceptance rate and solved status. Filter by difficulty,
  tag, status; search by title. Backed by LeetCode's GraphQL endpoint.
- **Language choice per problem.** Java and Python first, with the language
  list structured as data so adding more is a table entry, not a code change.
- **Real-file workspaces.** Each language gets a directory that a language
  server recognises as a project:

  ```
  ~/.leet-mode/workspace/
    python/           pyproject.toml, solutions/two_sum.py
    java/             pom.xml, src/main/java/TwoSum.java
  ```

  Java additionally needs the public class name to match the file name, so
  slug-to-identifier conversion is part of the template layer, not an
  afterthought.
- **Run and submit.** Send the buffer's contents to LeetCode, poll the judge,
  and render the verdict in its own buffer: status, runtime, memory, and on
  failure the failing input, expected output and actual output.

## Non-goals (for now)

- Local execution of test cases. The remote judge is the source of truth.
- Contests, discussions, editorial content.
- Supporting every language LeetCode offers before Java and Python feel good.

## First steps

1. **Auth spike.** Everything else depends on holding a valid session cookie,
   so prove this path before building anything on top of it. Determine which
   browser to drive and how its cookies are readable:
   - `xwidget-webkit` inside Emacs — its WebKit store is a readable sqlite file;
     best outcome, since login never leaves the editor.
   - Firefox — `cookies.sqlite` is plaintext.
   - Chrome on macOS — cookie values are AES-encrypted with a key stored in
     Keychain (`security find-generic-password -s "Chrome Safe Storage"`).
   Deliverable: a function that returns a live session cookie, and a manual
   paste fallback that stores it in authinfo.
2. **GraphQL layer.** A thin request wrapper that attaches the session cookie
   and CSRF token. Fetch the problem list and a single problem's detail
   (description, code snippets per language, sample tests).
3. **Problem list buffer.** Render the fetched list, with filtering.
4. **Workspace + templates.** Given a problem and a language, materialise the
   right directory and file from LeetCode's own code snippet, then open it.
   Confirm `eglot` attaches unprompted in both Python and Java.
5. **Submit and verdict.** Submission request, judge polling, result buffer.
6. **Polish.** A transient or prefix keymap, description rendered as readable
   text, solved-state refresh after a successful submission.

## Status

Nothing implemented yet. This README is the plan.
