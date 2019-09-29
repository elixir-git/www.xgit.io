---
title: It's All About the Trees
sub_title: Building up git tree data structures.
last_modified_at: 2019-09-29T08:37:17-07:00
---

The last few weeks have been devoted to understanding and constructing git `tree` data structures.

As I mentioned at the end of the last article here, the next logical section in _Git Internals_ would be to explore [Tree Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects#_tree_objects). That work is now largely completed (albeit with a few new [TO DO issues](https://github.com/elixir-git/xgit/issues?q=is%3Aissue+is%3Aopen+label%3A%22TO+DO%22) generated).

## Latest Releases

There were three new releases in September:

* [v0.2.2](https://github.com/elixir-git/xgit/releases/tag/v0.2.2): Implemented [`Xgit.Plumbing.WriteTree`](https://hexdocs.pm/xgit/Xgit.Plumbing.WriteTree.html#content). This is an API equivalent to [`git write-tree`](https://git-scm.com/docs/git-write-tree).

* [v0.2.3](https://github.com/elixir-git/xgit/releases/tag/v0.2.3): Implemented [`Xgit.Plumbing.CatFile.Tree`](https://hexdocs.pm/xgit/Xgit.Plumbing.CatFile.Tree.html#content). This is an API equivalent to [`git cat-file -p`](https://git-scm.com/docs/git-cat-file#Documentation/git-cat-file.txt--p) when the target object is of type `tree`.

* [v0.2.4](https://github.com/elixir-git/xgit/releases/tag/v0.2.4): Implemented [`Xgit.Plumbing.ReadTree`](https://hexdocs.pm/xgit/Xgit.Plumbing.ReadTree.html#content). This is an API equivalent to [`git read-tree`](https://git-scm.com/docs/git-read-tree).

## Offline First Camp

As I write this, I am preparing for Day 2 of [Offline First Camp](http://offlinefirst.org/camp/) near Grants Pass, Oregon. This is my second time attending Offline First camp. Offline First is a movement that believes applications (web and otherwise) should work well when clients and servers may be distributed and sporadically connected.

Xgit builds upon that vision by suggesting that git could be a backbone for transmitting and managing data in such an environment and by providing a server-friendly implementation of git.

It's been fun to spend this weekend with other people who share this mindset and to hear the parts of the offline-first challenge that they are thinking about and working to solve.

## What's Next?

With the work on tree data structures largely completed, I'll be proceeding to the next section in _Git Internals_, which will have me working on [Commit Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects#_git_commit_objects).
