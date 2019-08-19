---
title: "All the World’s a (git) Stage"
sub_title: "Two new releases in the last two weeks."
last_modified_at: 2019-08-16T07:43:58-07:00
---

It's been a big two weeks in Xgit land!

When I [last posted](/burning-down-the-house/), I had just made the first public release of the from-the-ground-up version of Xgit.

Since then, there have been two significant releases and two others that were twiddling bits trying to get the documentation to a happy place.

In these two releases, I doubled down on a couple of development patterns that I like:

* **Make modules (and files) as small as possible.** A module should have one clear purpose and do that one thing well. You'll find that Xgit already has many small modules and correspondingly many test scripts. Further, when a git command has multiple forms that have essentially incompatible argument sets, I break those into independent modules. You'll see an example of that later in this article.

* **Make releases as small and as frequent as possible.** Each of the two releases I called "significant" introduce _one_ new developer-facing feature to Xgit. (A "developer-facing feature" is a _plumbing_ or _porcelain_ command.) I take that feature and drill down recursively to implement whatever is necessary to make that top-level feature possible. (You'll also find that I apply the same logic to commits to the master branch.)

## So What's New?

### `git cat-file`

I'll talk to the first of the releases since it's the smaller one.

I had previously implemented `git hash-object`, which wrote a single object (blob, tag, tree, commit) into the loose object store.

In the [0.1.1 release](https://github.com/elixir-git/xgit/blob/master/CHANGELOG.md#v011), I implemented the inverse of that: [`git cat-file`](https://git-scm.com/docs/git-cat-file), which finds an object and returns its type, size, and content. For now, since Xgit doesn't understand pack files, it can only read from the loose object store.

This command is implemented as [`Xgit.Plumbing.CatFile`](https://hexdocs.pm/xgit/Xgit.Plumbing.CatFile.html).

In this release, I also changed the pattern of error responses from `{:error, "reason"}` to `{:error, :reason}` and added `@spec` documentation to call out all possible error responses.

### `git ls-files --stage`

As I mentioned in the previous post, I'm reading the [**git objects** chapter of the git internals book](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects), implementing each portion in Elixir as I read it. I'm now working through the portion titled _[Tree Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects#_tree_objects)._

After `hash-object` and `cat-file`, the next command that is discussed is `git update-index --add`. I started to implement that, but realized that it involved writing code for _both_ reading and writing the `.git/index` file (and creating abstractions for both actions at several layers).

In the "keep it simple" spirit, I decided to build only the reading side of it. That command is [`git ls-files --stage`](https://git-scm.com/docs/git-ls-files#Documentation/git-ls-files.txt---stage).

Given that `ls-files` has many distinct flavors and response patterns, depending on how it is invoked, I decided to split this apart more finely than `git` itself does.

I implemented _only_ the version that lists the contents of the index file, and it is known in Xgit as [`Xgit.Plumbing.LsFiles.Stage`](https://hexdocs.pm/xgit/Xgit.Plumbing.LsFiles.Stage.html).

This involved creating new abstractions at several levels. I'll walk through them from top to bottom:

* [`Xgit.Plumbing.LsFiles.Stage`](https://hexdocs.pm/xgit/Xgit.Plumbing.LsFiles.Stage.html), which implements the developer-facing API.

* [`Xgit.Repository.WorkingTree`](https://hexdocs.pm/xgit/Xgit.Repository.WorkingTree.html), which implements the on-disk manifestation of a working tree. A larger concept of working tree (index file and checked-out file content) will evolve here; only the `.git/index` file support is implemented in this version.

* [`Xgit.Repository.WorkingTree.ParseIndexFile`](https://hexdocs.pm/xgit/Xgit.Repository.WorkingTree.ParseIndexFile.html), which specifically reads the version of the `.git/index` file that I have encountered most frequently (version 2).

* [`Xgit.Core.DirCache`](https://hexdocs.pm/xgit/Xgit.Core.DirCache.html), which provides an abstract concept of a directory cache, independent from any specific file format.

This distinction points up an important piece of the design work inherent in a project like this: **drawing abstractions at the right places.** It was initially tempting to create a `DirCache` module that did all the things. Doing so would have been at cross-purposes to a core goal of Xgit: **Storage of a git repository need not be file-system based.** Doing the extra work of splitting out the file format parsing from the _concept_ of a directory cache better supported that goal.

All of this is now available on [Hex](https://hex.pm/packages/xgit) and [HexDocs](https://hexdocs.pm/xgit/0.1.4/Xgit.html) as version **0.1.4**.

## Help Wanted

There are a handful of [open issues](https://github.com/elixir-git/xgit/issues) in GitHub. While I'm always grateful for any attention to any of those issues, many of them are just "to do" list items that I've punted on in the interest of getting into deeper and more interesting topics.

There are a few issues that are particularly challenging. Some feedback from people with deeper git or Elixir knowledge would be helpful.

* **Elixir/OTP design advice wanted** – _[Issue #88: `Xgit.Repository.WorkingTree.dir_cache/1` output is potentially large](https://github.com/elixir-git/xgit/issues/88)._ This function returns a full copy of the `Xgit.Core.DirCache` struct, complete with all of its entries. That's fine for a small repo, but risky for a potentially large repo, where the number of entries is potentially unbounded. I'd love advice on how to pass back a _snapshot_ of the `DirCache` struct without overwhelming the VM message-passing mechanism.

* ~~**Git internals knowledge wanted** – _[Issue #67: There is content at the end of a `.git/index` file that I don't understand](https://github.com/elixir-git/xgit/issues/67)._ I've read the [git index format](https://github.com/git/git/blob/master/Documentation/technical/index-format.txt) specification closely. Every index file I've been able to construct comes up as a version 2 file and the index _entries_ match the specification as written. But what follows the entries (where I would _expect_ to find the **extensions** portion) makes no sense. Where I would _expect_ to to see the first extension signature, there is instead a 32-byte block of data that I can't comprehend. If someone knows what is actually being written there, I'd love to hear about it.~~ (Update: I've since figured this out.)

* **Erlang `zlib` library knowledge wanted** – _[Issue #50: No code coverage for zlib `:continue` case](https://github.com/elixir-git/xgit/issues/50)._ The documentation for function [`:zlib.safeInflate/2`](http://erlang.org/doc/man/zlib.html#safeInflate-2) says that the function may return `{:continue, data}`, but I have not been able to construct any test data for which that occurs. If someone knows how to trigger that …

If you have knowledge that can help – especially on these three issues – please respond in comments here, on Elixir Forum, or in the GitHub issues themselves. Thank you!

## What's Next

Having now taught Xgit how to _read_ index files, I now want to teach it how to _write_ them. So, as I have time coming up, I'll be working on an implementation of [`git update-index`](https://git-scm.com/docs/git-update-index).

Thank you for taking the time to follow along!
