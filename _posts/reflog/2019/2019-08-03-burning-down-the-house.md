---
title: "Burning Down the House"
sub_title: "To port or not to port?"
last_modified_at: 2019-08-03T10:24:00-07:00
---

To port, or not to port?

That is the question.

This project started in February of this year as a port of [jgit](https://www.eclipse.org/jgit/), a 100% native-Java implementation of git. I understood git to be a deep and complex system. I thought the planning that went into jgit would help me build an appropriate architecture for an Elixir-native implementation of git. I could work my way up the dependency tree of Java classes and build comparable mechanisms in Elixir and – voilà – I'd have xgit!

That mindset actually held up for quite a while.

Last month I got stuck figuring out how to get a toehold in porting the jgit code for walking revision trees.

That led to a stunning realization …

**Elixir and Java are not the same.**

The two language environments have fundamentally different approaches to system design. I **like** Elixir precisely _because_ it isn't object-oriented, _because_ it's functional, _because_ it has immutable data, _because_ it uses process boundaries to forcibly hide implementation details.

Java and the JVM are none of those things.

Jgit is a fantastic implementation of git given the design space and capabilities of the Java runtime. But it does a poor job of describing how you would build git in a purely-functional world.

And so, with love, I decided to part ways with the jgit port.

The remains of that project are available as [elixir-git/archived-jgit-port](https://github.com/elixir-git/archived-jgit-port/) on GitHub, but I do not plan to continue work on that version of the project.

I don't regret having done the port. It informs the work I'm now doing, which is a from-the-ground-up native-Elixir approach to git. There are pieces of that work that will live on in the new effort. And I understand some of the key concepts in ways that I hadn't before I started that port.

## Meet the New Xgit

I'm taking a completely different approach to this version of Xgit. I'm reading the [_Git Internals_](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain) chapter of the Pro Git book and building implementations of the concepts described there page-by-page.

For now, I'm just on _plumbing_ commands – the inner workings of git that form the building blocks for larger concepts (add, commit, branch, etc. – the _porcelain_ commands) that we're familiar with as end-users of git.

A primary design goal of Xgit is to allow git repositories to be stored in arbitrary locations other than file systems. (In a server environment, it likely makes sense to store content in a database or cloud-based file system such as S3.)

For that reason, the concept of **"repository"** in Xgit is kept intentionally minimal. [`Xgit.Repository`](https://hexdocs.pm/xgit/Xgit.Repository.html) is a behaviour module that describes the interface that a storage implementor would need to implement and very little else. (A repository is implemented using `GenServer` so that it can maintain its state independently. [`Xgit.Repository`](https://hexdocs.pm/xgit/Xgit.Repository.html) provides a wrapper interface for the calls that other modules within Xgit need to make to manipulate the repository.)

A **typical end-user developer** will typically construct an instance of [`Xgit.Repository.OnDisk`](https://hexdocs.pm/xgit/Xgit.Repository.OnDisk.html) (or some other module that implements a different storage architecture as described next) and then use the modules in the [`api` folder](https://github.com/elixir-git/xgit/tree/master/lib/xgit/api) folder to inspect and modify the repository.

A **storage architect** will construct a module that encapsulates the desired storage mechanism in a `GenServer` process and makes that available to the rest of Xgit by implementing the [`Xgit.Repository`](https://hexdocs.pm/xgit/Xgit.Repository.html) behaviour interface. I am building the [`Xgit.Repository.OnDisk`](https://hexdocs.pm/xgit/Xgit.Repository.OnDisk.html) module, which demonstrates the _concept_ of a storage architecture, but my hope is that other developers could implement other storage mechanisms by providing new implementations of the [`Xgit.Repository`](https://hexdocs.pm/xgit/Xgit.Repository.html) behaviour.

## Categories of Code

Code in Xgit is organized into the following categories, reflected in module naming and corresponding folder organization, listed here roughly in order from top to bottom of the dependency sequence:

* [**`api`**](https://github.com/elixir-git/xgit/tree/master/lib/xgit/api) _(none implemented yet)_: These are the typical commands or operations that you perform on a git repository. In the git vernacular, these are often referred to as **porcelain** (i.e. the refined, user-visible operations).

* [**`plumbing`**](https://github.com/elixir-git/xgit/tree/master/lib/xgit/plumbing): These are the raw building-block operations that are often composed together to make the user-targeted commands. These are often sophisticated operations in and of themselves, but are typically not of interest to end-user developers.

* [**`repository`**](https://github.com/elixir-git/xgit/tree/master/lib/xgit/repository): This describes how a single git repository is persisted. A reference "on-disk" implementation is provided and is designed to interoperate with the existing git command-line tool.

* [**`core`**](https://github.com/elixir-git/xgit/tree/master/lib/xgit/core): The modules in this folder describe the fundamental building blocks of git's data model (objects, object IDs, tags, commits, etc.). These are used within Xgit to communicate about the content in a repository.

* [**`util`**](https://github.com/elixir-git/xgit/tree/master/lib/xgit/util): The modules in this folder aren't really part of the data model _per se_, but provide building blocks to make higher layers of Xgit possible.

## Starting Points

Two git commands have been implemented thus far:

### [`git init`](https://git-scm.com/docs/git-init)

A minimal implementation of `git init` has been made available as part of the reference (on-disk) implementation of repository.

See [`Xgit.Repository.OnDisk.create/1`](https://hexdocs.pm/xgit/Xgit.Repository.OnDisk.html#create/1) for details. This is, by far, not a complete port of `git init`. There are many edge cases that are left unimplemented. But it's enough to allow us to bootstrap further work.

### [`git hash-object`](https://git-scm.com/docs/git-hash-object)

This is the first of the commands referenced in the [**git objects** part of the git internals book](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects).

See [`Xgit.Plumbing.HashObject.run/2`](https://hexdocs.pm/xgit/Xgit.Plumbing.HashObject.html#run/2) for details.

## Testing Strategy

I may be stating the obvious, but a key goal of Xgit must be to interoperate correctly with other implementations of git.

People who know me from my day job know that I am famous for this saying:

> What you don't test, you will ^@(* up.

So it should surprise no one that I'm employing a very aggressive strategy of verifying interoperability. Many of the tests for the above two commands run an operation in Xgit using the on-disk repository implementation and run the same operation again using command-line git. If the same operation has been done in both versions, the folder contents should be identical. If they are, the test passes.

## What's Next?

Having wrapped up `git hash-object`, I'll continue through the [git objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects) page. Next up is `git cat-object`.
