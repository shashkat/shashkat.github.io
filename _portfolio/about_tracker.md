---
title: "about-tracker: Metadata for Files and Directories"
excerpt: "A command line tool to attach markdown descriptions to files and directories, and see them right in your ls output.<br/><img src='/images/about_tracker/demo_600w.gif'>"
collection: portfolio
---

*Mar 29, 2026*

## About:

about-tracker is a small command line tool I wrote to add and manage metadata (descriptions) for files and directories. The idea came from a problem I kept running into in my own work: a file or directory name is usually the only place to record what something is, so names end up carrying far more than they should. Something like-

```
results_final_v2_fixed_lr0.001_USE_THIS.csv
experiment_3_(same as 2 but new seed, broken - dont use)/
```

Names like these are hard to read, awkward to type in a terminal, and still leave out the context that matters most: why the file exists, where it came from, and whether you should use it. Keeping a separate notes file doesn't help much either, because nobody opens it, and it goes out of date as soon as files get renamed or moved.

about-tracker lets the names stay short, and puts the description right next to the file instead. In simple terms, it works as follows-

- **Short names, full descriptions:** `abt modify results.csv` opens a small markdown file (`.about_results.csv.md`) in your editor, where you can write as much as you want about that file.
- **Descriptions are shown where you already look:** `abt ls` prints the normal `ls` output, followed by the description of the current directory and of every entry inside it.
- **Descriptions follow their files:** `abt mv`, `abt cp` and `abt rm` move, copy and remove the description along with the file. And if a file was moved some other way (for example in Finder), `abt doctor` finds the descriptions that were left behind.
- **Plain files, no lock-in:** the descriptions are just hidden markdown files, so they can be read without about-tracker, committed to git, and synced like any other file.

I personally use it with `ls`, `cp`, `mv` and `rm` aliased to their `abt` versions, so that the descriptions just always show up without me having to think about it.

## Example:

Below is what the output of `abt ls` looks like in a directory with a few described entries:

![abt ls output example](/images/about_tracker/ls_example_ss.png)

And here is a short demo of adding descriptions and moving files around with about-tracker:

![about-tracker demo](/images/about_tracker/demo.gif)

## Implementation:

The whole tool is written in shell script, so it has no dependencies to install and works in both zsh and bash. There is a single `abt` entry point which dispatches to one script per subcommand (`ls`, `cp`, `mv`, `rm`, `modify` and `doctor`), with shared logic kept in a common functions file. It comes with install and uninstall scripts which take care of setting up (and cleanly removing) the shell configuration, and the uninstall never touches your metadata files. The behaviour of each command is covered by tests written with [bats-core](https://github.com/bats-core/bats-core), which run against temporary copies of test fixtures so that they never change anything on your actual system.

If you have any questions or suggestions, please feel free to email me or raise an issue on the repo. I would be very happy to hear if you find it useful!

## Links:
- github link to project: *[https://github.com/shashkat/about-tracker](https://github.com/shashkat/about-tracker)*
