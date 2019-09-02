---
title: Judging a Bug by Its Cover
sub_title: On index files and code coverage
last_modified_at: 2019-09-01T21:26:47-07:00
---

On index files and code coverage …

## On the Shape of a Revision Graph

I've noticed a recurring pattern in the shape of the revision graph when I'm working on Xgit. In [SourceTree](https://www.sourcetreeapp.com), which is how I work with git on my Mac laptop, it looks like this:

![rev-graph](/assets/images/posts/2019/2019-08-25-rev-graph-shape.png)

_What I think that says about my working style:_ I tend to sketch out the top-level API and its test suite first -- knowing full well that it will be missing some important pieces -- just to find out what it feels like to work with that API. I'm essentially asking myself, "What _should_ this API look like? Would I enjoy working with it in this form?"

Invariably, that process leads to discovering new requirements at a layer below that and that may happen a few times. So it's basically a recursive descent through implementing that one thing.

The on-line book [_Shape Up_](https://basecamp.com/shapeup) from the team at Basecamp has been making the rounds at my work. This recursive descent to implement one small feature feels a lot like what they talk about in Chapter 10, ["Get One Piece Done"](https://basecamp.com/shapeup/3.2-chapter-10): Do a small thing, do it through all the layers, be ready to show it off, lather, rinse, repeat.

## So … What's New This Cycle?

Since I last posted, there have been several small releases of Xgit.

To wit:

### 0.1.5: The Curious Case of the 32 Bytes at the End of the Index File

In my [last post](/all-the-worlds-a-git-stage/#help-wanted), I wrote that there was content at the end of the `.git/index` file that I could not understand. I learned a rather ~~embarrassing~~ \*cough\* _amusing_ lesson here: **Be careful what tools you use to read files.**

It turns out that the tool I had been using to introspect the `.git/index` file I had constructed was silently translating the file from Latin-1 to UTF-8 encoding. In so doing, the 20-byte SHA-1 hash at the end of the index file puffed up to 32 bytes. If you interpret all-but-the-last 20 bytes as standard file content, that left 12 bytes of "extension" data, but there was nothing resembling an extension signature in that position. Hilarity ensued.

Anyhow, I've since discovered `xxd` which does a hex dump of an _untranslated_ binary file -- thank you -- and have updated the index file parsing code to understand and verify that trailing SHA-1 signature. Xgit does not yet read or write extensions in index files. I'll address that as the need arises.

In that same post, I also wrote that I was seeking help on `zlib` and how to generate coverage for a particular case of `safeInflate`. I [got the help I needed](https://github.com/elixir-git/xgit/issues/50#issuecomment-522357816), and Xgit is now back to 100% code coverage.

Those two issues and one other minor performance improvement formed the core of the 0.1.5 release.

### 0.1.6: Update Index File

I then spent the next week or so building the API equivalent of `git update-index --cacheinfo`, which I call [`Xgit.Plumbing.UpdateIndex.CacheInfo`](https://hexdocs.pm/xgit/0.1.6/Xgit.Plumbing.UpdateIndex.CacheInfo.html#content). The rev graph above was captured during that time. Xgit is now able to both read and write index files and modify [`Xgit.Core.DirCache`](https://hexdocs.pm/xgit/0.1.6/Xgit.Core.DirCache.html#content) structures accordingly.


### 0.2.0: Refactoring

After this, I started to realize that some code was scattered in places that didn't feel right to me. Specifically, I had two pairs of modules that were essentially talking about the same concept. (I almost typed "object" there. Please forgive me.)

Some of that was left over from the earlier port from jgit. In that effort, I was attempting to follow the structure of jgit closely in hopes that the pieces would all fit together neatly. As I [wrote a month ago](/burning-down-the-house/), that didn't happen. So now that I am owning the architecture of Xgit as a from-the-ground-up native Elixir application, I can -- and did -- let go of those vestiges.

These got cleaned up in:

* **file paths** ([PR 129](https://github.com/elixir-git/xgit/pull/129))
* **objects** ([PR 132](https://github.com/elixir-git/xgit/pull/132))

It did, of course, break API compatibility, so that's why we now have an 0.2 release. Semver makes it look like a big advance, but really, it isn't.

### 0.2.1: Uncovering a Coverage Issue

I posted on [Elixir Forum](https://elixirforum.com/t/save-my-from-myself-code-coverage-misses-lines-that-return-a-literal/24931) a few weeks ago about a problem with code coverage in Elixir. (Maybe it's specific to [excoveralls](https://github.com/parroty/excoveralls/issues/146), but I suspect the issue is larger than that.)

Specifically, lines of code that return literal values do not get flagged as executable. This also applies to _some_ tuples that contain variables, but I haven't figured out the exact pattern yet.

As an example, consider the following passage from `Xgit.Core.Object`:

```elixir
defp check_commit(%__MODULE__{content: data}) when is_list(data) do
  with {:tree, data} when is_list(data) <- {:tree, after_prefix(data, 'tree ')},
       {:tree_id, data} when is_list(data) <- {:tree_id, check_id(data)},
       {:parents, data} when is_list(data) <- {:parents, check_commit_parents(data)},
       {:author, data} when is_list(data) <- {:author, after_prefix(data, 'author ')},
       {:author_id, data} when is_list(data) <- {:author_id, check_person_ident(data)},
       {:committer, data} when is_list(data) <- {:committer, after_prefix(data, 'committer ')},
       {:committer_id, data} when is_list(data) <- {:committer_id, check_person_ident(data)} do
    :ok
  else
    {:tree, _} -> {:error, :no_tree_header}
    {:tree_id, _} -> {:error, :invalid_tree}
    {:parents, _} -> {:error, :invalid_parent}
    {:author, _} -> {:error, :no_author}
    {:author_id, why} when is_atom(why) -> {:error, why}
    {:committer, _} -> {:error, :no_committer}
    {:committer_id, why} when is_atom(why) -> {:error, why}
  end
end
```

If you look at the [coverage report from version 0.2.0](https://coveralls.io/builds/25412181/source?filename=lib/xgit/core/object.ex#L225), _none_ of the error exits nor the `:ok` exit path are marked as executable.

_That's a lot of code that **might** not be tested._

Those of us who pay attention to code coverage and _branch_ coverage feel kind of naked without proof that our code is being tested.

So [I hacked my way to more coverage](https://github.com/elixir-git/xgit/pull/122.). I'll repeat what I said on Elixir Forum: **I don't like this hack.** But I like even less the idea that I have to sacrifice insight into how thorough I've been in my unit testing because of this issue

Here are some interesting metrics: In the previous (0.2.0) release, there were [604 executable ("relevant") lines](https://coveralls.io/builds/25412181) of code in Xgit. When I merged this PR, that number [went up to 913](https://coveralls.io/builds/25469915).

**I was missing insight on roughly 1/3 of my code.** That's a problem.

I would _love_ for Elixir to address this problem inherently in the language. But until then, I'm gonna use my `cover` macro hack.

## What's Next for Xgit?

This message comes as we in the United States celebrate our Labor Day holiday. Like many Americans, I'm off celebrating and not really focusing on code.

When I get back to it (soon!), the next logical section in _Git Internals_ would be to explore [Tree Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects#_tree_objects). At a glance, this will require:

* understanding references (branches)
* parsing `tree` and possibly `commit` object types

Thanks again for your interest in this project. We'll see you again soon.
