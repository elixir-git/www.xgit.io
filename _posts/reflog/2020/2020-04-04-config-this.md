---
title: Config This
sub_title: Config kicked my butt
last_modified_at: 2020-04-04T22:55:07-07:00
---

So, yes, it's been a while.

Life happens, both on a global scale and on a personal scale.

I am fortunate. I am healthy, and so far everyone in my immediate family is as well.

I have a new day job – for which I am deeply honored and thrilled – as lead developer for Adobe's portion of the [Content Authenticity Initiative](https://contentauthenticity.org).

And I moved a couple of months ago. So, as you might imagine, personal life, work life, and the more recent global situation have demanded more attention than usual.

Xgit is a labor of love.

Sometimes such efforts need to take a back seat to more urgent priorities for a time. I think I'm out of the woods that way. I've been able to devote some time here and there on weekends to continuing the effort; I'm hopeful that this can be sustainable for now.

## Config Kicked My Butt

The most recent thing I've been building is the configuration system. (Think [`git config`](https://git-scm.com/docs/git-config).)

This turned out to have more design challenges than I anticipated. My first cut of the abstract storage engine was, frankly, a bit over-designed. A closer study of how `git config` actually works resulted in a simpler design that I'm now happy with.

## Latest Releases

There have been several new releases since mid-December, when I last wrote here:

* [v0.7.0](https://github.com/elixir-git/xgit/releases/tag/v0.7.0): Implemented [`Xgit.Repository.Plumbing.delete_symbolic_ref/2`](https://hexdocs.pm/xgit/0.7.0/Xgit.Repository.Plumbing.html#delete_symbolic_ref/2). This is an API analogue for [`git symbolic-ref --delete (ref_name)`](https://git-scm.com/docs/git-symbolic-ref). Added a few other changes regarding symbolic refs that were considered API breaking.

* [v0.7.1](https://github.com/elixir-git/xgit/releases/tag/v0.7.1): Implemented [`Xgit.Repository.Plumbing.cat_file_tag/2`](https://hexdocs.pm/xgit/0.7.1/Xgit.Repository.Plumbing.html#cat_file_tag/2). This is an API equivalent to [`git cat_file -p`](https://git-scm.com/docs/git-cat-file#Documentation/git-cat-file.txt--p) when the target object is of type `tag`.

* [v0.7.2](https://github.com/elixir-git/xgit/releases/tag/v0.7.2): Implemented [`Xgit.Repository.tag/4`](https://hexdocs.pm/xgit/0.7.2/Xgit.Repository.html#tag/4). This is an API equivalent to the creation case of [`git tag`](https://git-scm.com/docs/git-tag).

* [v0.7.3](https://github.com/elixir-git/xgit/releases/tag/v0.7.3): Implemented [`Xgit.ConfigFile`](https://hexdocs.pm/xgit/0.7.3/Xgit.ConfigFile.html), which reads and writes git-style config files. Added support for Elixir 1.10.

* [v0.8.0](https://github.com/elixir-git/xgit/releases/tag/v0.8.0): Implemented [`Xgit.Config`](https://hexdocs.pm/xgit/0.8.0/Xgit.Config.html#content) module. This is roughly analogous to the [`git config`](https://git-scm.com/docs/git-config) command. This added new requirements to [`Xgit.Repository.Storage`](https://hexdocs.pm/xgit/0.8.0/Xgit.Repository.Storage.html#content) behaviour, and was thus API breaking.

## What's Next?

The next likely avenues of development are:

* [Remotes](https://git-scm.com/book/en/v2/Git-Internals-Git-References#_remotes)
* [Pack files](https://git-scm.com/book/en/v2/Git-Internals-Packfiles)
