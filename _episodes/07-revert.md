---
title: Recovering Old Versions
teaching: 10
exercises: 5
questions:
- "How can I recover old versions of files?"
objectives:
- "Restore older versions of files."
- "Use configuration aliases to create custom Mercurial commands."
keypoints:
- "Use `hg update` to examine old versions of files."
---

All right:
we can save changes to files and see what we've changed—how can we restore older versions of things?

## Inspecting and returning to earlier versions

Part of the point of carefully committing previous versions is so that
we can return to those versions later.  We might do this for reference
– to see how things stood some point in the past – or because we want
to make a separate set of developments to an earlier version.

To see what versions we have available, we can look at the history of
a repository, with `hg log`, which shows all of the committed versions
of the code.

~~~
changeset:   2:2e15a7ee29c2
tag:         tip
user:        Susan Allen <sallen@eos.ubc.ca>
date:        Tue Jun 09 15:28:25 2015 +0200
summary:     Add note about data source for Fraser River flow forcing.

changeset:   1:b31241913818
user:        Susan Allen <sallen@eos.ubc.ca>
date:        Tue Jun 09 15:16:11 2015 +0200
summary:     Add note about source for atmospheric forcing.

changeset:   0:1320339bbcae
user:        Susan Allen <sallen@eos.ubc.ca>
date:        Tue Jun 09 14:41:27 2015 +0200
summary:     Starting to plan the daily NEMO forecast system.
~~~
{: .output}

> This might end up a rather long list, so the command `hg
> log -l5` will list just the last five versions.  The command `hg log
> -Gl5` will do the same, and additionally show you, with a `@` sign, which of these
> versions is the ‘current’ one.
{: .callout}

We look at older snapshots by ‘updating’ to an earlier version with
the `hg update` command.  We can specify the version to update to
_either_ by giving the long identifier such as `b31241913818` or the
more convenient corresponding short index `1`.

~~~
$ hg update -r1
~~~
{: .bash}

~~~
1 files updated, 0 files merged, 0 files removed, 0 files unresolved
~~~
{: .output}

This changes the files in your working directory, to the state they
were in when you did commit number 1.

~~~
$ cat plan.txt
~~~
{: .bash}

~~~
Goal: Run NEMO everyday to forecast storm surge water levels

Need daily high resolution weather forcing from Environment Canada.
~~~
{: .output}

If you were to modify the files in this version, and commit, you would
‘branch’ the repository, and have two separate lines of development
from here.  This can be a useful thing to do if you want two related
but distinct versions of your project, or if you want to abandon a
failed experiment, and restart from this point in the past, without losing track of the
details.  Though useful, this can get confusing, so we will mention it
only in passing, and instead note that you can get back to the most
recent version using `hg update` as well:

~~~
$ hg update
~~~
{: .bash}

~~~
1 files updated, 0 files merged, 0 files removed, 0 files unresolved
~~~
{: .output}

~~~
$ cat plan.txt
~~~
{: .bash}

~~~
Goal: Run NEMO everyday to forecast storm surge water levels

Need daily high resolution weather forcing from Environment Canada.
Also need daily average Fraser River flow from Environment Canada.
~~~
{: .output}

## Discarding changes

Let's suppose we (somehow) accidentally overwrite Salish Sea NEMO forecast
planning file with our grocery list:

~~~
$ nano plan.txt
$ cat plan.txt
~~~
{: .bash}

~~~
Ricotta
Mushroom Tortellini
Bacon
~~~
{: .output}

`hg status` now tells us that the file has been changed,
but those changes haven't been committed:

~~~
$ hg status
~~~
{: .bash}

~~~
M plan.txt
~~~
{: .output}

We can put things back the way they were by using `hg revert`:

~~~
$ hg revert plan.txt
$ cat plan.txt
~~~
{: .bash}

~~~
Goal: Run NEMO everyday to forecast storm surge water levels

Need daily high resolution weather forcing from EC.
Also need daily average Fraser River flow from EC.
~~~
{: .output}

As you might guess from its name,
`hg revert` reverts to (i.e., restores) an old version of a file.
In this case,
we're telling Mercurial that we want to recover the last committed version
of the file.
We'd typically do this if we realise that we've messed up our
modifications to a file, and want to revert to the last-known-good
version which will (because we've been carefully committing good
versions) be the last revision of this document.  Thus `hg revert` is
a ‘rip it up and start again’ action.

When you (occasionally) use this command, you'll typically do so to a
single file.  If it's all gone horribly wrong, there's a `hg revert
--all` option.

You might _sometimes_ find it useful to revert a file to a particular
known-good version, and you can do that with `hg revert -r1
plan.txt`.  After that, Mercurial regards that file as having
‘changed’, and if you want to keep this version, you will have to commit.

Mercurial really doesn't want to cause us to lose our work,
so it defaults to making a backup when we use `hg revert`:

~~~
$ hg status
~~~
{: .bash}

~~~
? plan.txt.orig
~~~
{: .output}

The `plan.txt.orig` file is a copy of `plan.txt` as it stood before the
`hg revert` command.
It's not tracked by Mercurial.
It's just there in case we made a mistake and really didn't want to revert,
or in case there's some content from before the revert that we decide that
we really do want to copy into `plan.txt`.
When we're sure that we don't need `*.orig` files we can just go ahead and
delete them.

> ## Using Aliases to Create New Mercurial Commands
>
> You might find yourself typing a command like `hg log -Gl5` very
>frequently.  In that case we can use `hg config --edit` to add
>
> ~~~
> [alias]
> l5 = log -Gl5
> ~~~
> {: .source}
>
> to our Mercurial configuration settings.
> After we save the configuration file and exit from the editor we have a new
> command,
> `hg l5`
> that is the same as typing `hg log -Gl5`.
> Note that it is good practice when creating aliases for commands to give them
> your own names instead of using the alias to redefine the built-in command.
> If we had used the alias to change add the `-l5` flag to the `log`
> command
>
> ~~~
> [alias]
> log = log -l5
> ~~~
> {: .source}
>
> it would be difficult to get `hg log` to produce its original output
> if we ever wanted it to.
{: .callout}
