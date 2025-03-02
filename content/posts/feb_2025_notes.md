---
title: Dev notes from Feb 2025
date: 2025-02-28
img:
---

### Git performance tracing

I noticed that some git commands, especially `git status`, were taking a long time to execute. Since it's tied to my command prompt, this also slowed down my prompt. By using `GIT_TRACE=1`, I could see what git was doing. The issue was that the repository had too many submodules, causing `git status` to run on each one, which took time.

```bash
$ GIT_TRACE=1 git status
14:40:25.548035 git.c:476               trace: built-in: git status
14:40:25.550695 run-command.c:667       trace: run_command: cd themes/hello-friend-ng; unset GIT_PREFIX; GIT_DIR=.git git status --porcelain=2
14:40:25.550763 run-command.c:759       trace: start_command: /opt/homebrew/opt/git/libexec/git-core/git status --porcelain=2
14:40:25.557470 git.c:476               trace: built-in: git status --porcelain=2
On branch main
Your branch is up to date with 'origin/main'.

14:40:25.567125 run-command.c:667       trace: run_command: GIT_INDEX_FILE=.git/index git submodule summary --cached --for-status --summary-limit -1 HEAD
14:40:25.567200 run-command.c:759       trace: start_command: /opt/homebrew/opt/git/libexec/git-core/git submodule summary --cached --for-status --summary-limit -1 HEAD
14:40:25.573148 git.c:769               trace: exec: git-submodule summary --cached --for-status --summary-limit -1 HEAD
...
```

### Fish shell performance tracing

Similarly, the fish shell has a profiling feature:

```bash
       -p or --profile=PROFILE_FILE
              when fish exits, output timing information on all executed commands to the specified file.  This excludes time spent starting up and reading the configuration.

       --profile-startup=PROFILE_FILE
              Will write timing for fish startup to specified file.
```

The profile file looks like this:

```bash
Time    Sum     Command
115     10404135        > __fish_print_help fish
435     437     -> source /opt/homebrew/Cellar/fish/4.0.0/share/fish/functions/__fish_print_help.fish
2       2       --> function __fish_print_help --description "Print help message for the specified fish function or builtin" --argument-names item error_message...
2       2       -> switch $item...
1       21      -> if not test -e "$__fish_data_dir/man/man1/$item.1" -o -e "$__fish_data_dir/man/man1/$item.1.gz"...
20      20      --> not test -e "$__fish_data_dir/man/man1/$item.1" -o -e "$__fish_data_dir/man/man1/$item.1.gz"
2       2       -> set -l help
1       1       -> set -l format
0       0       -> set -l cols
1       72      -> if test -n "$COLUMNS"...
```

### [Avante.nvim](https://github.com/yetone/avante.nvim)
I started using Avante.nvim with Copilot, and the results were surprisingly good. I avoided using the RAG service because it spins up a local Docker container, which drains resources.

This setup was nearly perfect for working through [rustlings](https://github.com/rust-lang/rustlings) problems. The suggested answers were correct 9 out of 10 times. Paired with rust-lsp and Clippy, I completed the rustlings problems in a weekend.

This experience is very similar to using Cursor AI. Overall, I'm very pleased with the setup.

