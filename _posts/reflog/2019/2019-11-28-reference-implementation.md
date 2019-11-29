---
title: A Reference Implementation
sub_title: It's all about commits and references.
last_modified_at: 2019-11-28T20:41:49-08:00
---

Since we last spoke …

Here in the United States, we are celebrating our Thanksgiving holiday today. Among the many things I have to be thankful for, one is the opportunity to spend time deeply learning git through building the Xgit library.

Some other aspects of my life have required extra attention in the last month or two, so progress on Xgit has slowed but it has definitely not stopped.

When I last posted here, the next section in _Git Internals_ was to work through [Commit Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects#_git_commit_objects). I've done that and am now well through the next section: [References](https://git-scm.com/book/en/v2/Git-Internals-Git-References).

## Latest Releases

There have been three new releases in October and November. As usual, each release focuses around one "plumbing"-level API:

* [v0.2.5](https://github.com/elixir-git/xgit/releases/tag/v0.2.5): Implemented [`Xgit.Plumbing.CommitTree`](https://hexdocs.pm/xgit/0.2.5/Xgit.Plumbing.CommitTree.html#content). This is an API equivalent to [`git commit-tree`](https://git-scm.com/docs/git-commit-tree).

* [v0.3.0](https://github.com/elixir-git/xgit/releases/tag/v0.3.0): Implemented [`Xgit.Plumbing.CatFile.Commit`](https://hexdocs.pm/xgit/0.3.0/Xgit.Plumbing.CatFile.Commit.html#content). This is an API equivalent to [`git cat-file -p`](https://git-scm.com/docs/git-cat-file#Documentation/git-cat-file.txt--p) when the target object is a `commit` object. This was marked API breaking because some implementation details were marked private.

* [v0.4.0](https://github.com/elixir-git/xgit/releases/tag/v0.4.0): Implemented [`Xgit.Plumbing.UpdateRef`](https://hexdocs.pm/xgit/0.4.0/Xgit.Plumbing.UpdateRef.html#content). This is an API equivalent to [`git update-ref`](https://git-scm.com/docs/git-update-ref). This was marked API breaking because the behaviour contract for `Xgit.Repository` was extended to add reference-related operations.

## GitHub Universe and GitHub Actions

I had the honor of attending the [GitHub Universe](https://githubuniverse.com) conference a couple of weeks ago. It was great to see the vibrancy of Git and GitHub and to have a chance to meet with people building and supporting this ecosystem.

If there was one takeaway from this conference, it was that [GitHub Actions](https://github.com/features/actions) are the new hotness. (Yes, similar things have existed elsewhere for some time, but GitHub has the market- and mind-share these days.)

While at the conference, I retooled Xgit to use GitHub Actions as its primary build system. This prompted a change to using [CodeCov](https://codecov.io/gh/elixir-git/xgit) instead of [Coveralls](https://coveralls.io), as the Coveralls API is not well-suited to receiving data from GitHub Actions builds as of yet.

I'm finding the new build system substantially faster (thank you [GitHub Cache Action](https://github.com/actions/cache), among other things) and more reliable than the previous system. So far, I'm quite happy with the new system.

## A Tribute to Noms

I was recently introduced to [Noms](https://github.com/attic-labs/noms), which describes itself as …

> … a decentralized database philosophically descendant from the Git version control system.

I'm sad to read that ["nobody is working on this right now"](https://github.com/attic-labs/noms#status) since Noms appears to have been instigated for many of the same reasons that inspired Xgit.

We align on:

* Supporting [offline-first](http://offlinefirst.org) applications (or, as I've seen recently, [local-first](https://blog.acolyer.org/2019/11/20/local-first-software/) applications).
* Being versioned and auditable.
* Allowing synchronization in arbitrary patterns (i.e. peer-to-peer in addition to cloud-to-client).
* Allowing for extensible storage mechanisms (S3, etc.).

We differ on:

* Technology choices (Go vs Elixir).
* Whether we use git directly (Xgit) or are influenced by it (Noms). (Xgit provides a server-focused implementation of git and assumes client developers can use [`libgit2`](https://libgit2.org) or [`jgit`](https://www.eclipse.org/jgit/), as appropriate for the platform at hand.)
* Whether we provide structured-data support intrinsically (Noms) or not (Xgit). In Xgit, I consider this to be a separate, but closely related problem. I prefer to allow application developers to layer on their own structured-data and conflict-resolution mechanisms.

Regardless of which libraries thrive and which ones fall by the wayside, I hope the _concepts_ explored by Xgit and Noms live on.

## What's Next?

There are a few more reference-related operations and concepts to implement:

* Symbolic references (see [`git symbolic-ref`](https://git-scm.com/docs/git-symbolic-ref))
* [Tags](https://git-scm.com/book/en/v2/Git-Internals-Git-References#_tags)
* [Remotes](https://git-scm.com/book/en/v2/Git-Internals-Git-References#_remotes)
