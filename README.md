# do-review

A small custom helper script to quickly review Linux kernel patches using
either Sashiko or Claude/Codex. Run it from a kernel Git working tree with a
lore.kernel.org message ID or URL, a local commit range, or a single commit.
Codex is the default reviewer.

## Setup

Requires Python 3.11 or later, Git, and the chosen reviewer CLI. Reviewing
patches from lore also requires `b4`.

Edit the TOML `config` beside the script to set the Claude/Codex binary names
or paths. For Sashiko, also set `SASHIKO_API_KEY` and `SASHIKO_SETTINGS` to
your API key and settings file. Binary names are resolved through `PATH`;
paths may start with `~`.

## Usage

With `do-review` on your `PATH`, run from your kernel checkout:

```sh
do-review '<message-id>'       # Codex
do-review --claude '<lore-url>'
do-review --sashiko HEAD~3..HEAD
do-review HEAD                # Review one commit
```

For lore targets, the script fetches the full thread and available revisions,
selects the latest revision, and checks whether the series is already applied.
If it is missing, the script applies it with `b4 shazam` before starting the
review. It stops if it detects an older applied revision. Use a clean kernel
working tree; the script can add commits when reviewing lore patches.

For local commits, the script generates patch emails with `git format-patch`.
The requested commits must be reachable from `HEAD`. In both modes, it saves
the email context under Git's `do-review` directory and passes the exact
commit range and context location to the reviewer. `--search-depth N` adjusts
the applied-series lookup limit (default: 1000 recent commits).

## Review prompt

`review.md` is the custom prompt used by Claude and Codex. It asks for a
thorough review of correctness, concurrency, performance, regression risks,
fix validity, and alternative implementations, checking the actual source
and email discussion. It requests concise, inline LKML-style reply drafts
quoting the original patch emails, without modifying the source or sending
replies.

Sashiko uses its configured review settings instead of `review.md`; the script
supplies the commit range and email-thread context through its custom prompt.
