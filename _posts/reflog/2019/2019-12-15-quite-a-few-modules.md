---
title: That's Quite a Few Modules
sub_title: An unexpected code review.
last_modified_at: 2019-12-15T20:32:55-08:00
---

Sometimes the best code review feedback happens when you aren't explicitly asking for a review.

I mentioned last month that I attended GitHub Universe.

So what happened was …

I was showing Xgit's documentation to somebody at the conference on my laptop. He seemed quite excited by the concept and was going through this site and the documentation I had published at the time on HexDocs. And then he said five words that I couldn't put out of my head:

> That's quite a few modules.

Well, that rattled around in my brain for a while and it triggered some refactoring that landed in today's 0.6.0 release.

## What Changed?

There were two fundamental objectives in this refactoring:

* Make the module hierarchy flatter and more intuitive for typical end-user developers.
* Eliminate the single-function modules.

This played out as the following concrete changes:

* Rename `Xgit.Repository` to `Xgit.Repository.Storage`.
* Merge all the plumbing commands together into a single module.
* Introduce new `Xgit.Repository` module. This will contain the porcelain-level commands when they are implemented.
* Rename `Xgit.Core.*` modules to `Xgit.*`.
* Move index file format parsing into `Xgit.DirCache` module.

## Latest Releases

There have been two new releases in late November and early December:

* [v0.5.0](https://github.com/elixir-git/xgit/releases/tag/v0.5.0): Implement [`Xgit.Plumbing.SymbolicRef.Put`](https://hexdocs.pm/xgit/0.5.0/Xgit.Plumbing.SymbolicRef.Put.html#content). This is an API analogue to the 2-argument form of [`git symbolic-ref`](https://git-scm.com/docs/git-symbolic-ref). This was marked API breaking because it introduced a new requirement on the `Xgit.Repository` behaviour.

* [v0.6.0](https://github.com/elixir-git/xgit/releases/tag/v0.6.0): Implemented the refactoring described above. This was marked API breaking because several modules were renamed and functions moved from one module to another.

## What's Next?

There are a few more reference-related operations and concepts to implement:

* Symbolic references (see [`git symbolic-ref`](https://git-scm.com/docs/git-symbolic-ref)) _(now partially implemented)_
* [Tags](https://git-scm.com/book/en/v2/Git-Internals-Git-References#_tags)
* [Remotes](https://git-scm.com/book/en/v2/Git-Internals-Git-References#_remotes)
