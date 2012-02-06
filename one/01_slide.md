!SLIDE 
# Git Core Concepts #

### by Ted Naleid ###

### http://naleid.com ###

!SLIDE 
# tl;dr #
rewriting history is a lie

!SLIDE 
# tl;dr #
git commits are immutable and cannot be "rewritten"

!SLIDE 
# tl;dr #
history is add-only with eventual garbage collection on unreferenced commits

!SLIDE center
# git's the safest VCS I've used #

![airtravel](img/airtravel.png)


!SLIDE center
# you just have to understand a few things #

![instrumentpanel](img/instrumentpanel.png)

!SLIDE center
# if you don't…it gets ugly fast #

![plane wreck](img/planewreck.png)

!SLIDE 
you need to throw away your preconceptions from other version control systems

!SLIDE 
git doesn't help by having a terrible UX

!SLIDE 
git mislabels things in confusing ways

<div class="small centered">
    ex: git branches aren't branches
</div>

!SLIDE 
git has hundreds of commands, but commonly used ones often require additional parameters

!SLIDE 
uses scary terms that all sound dangerous:

<div class="centered smaller">
<p>"rewrite history"</p>

<p>rebase</p> 

<p>reset --hard HEAD</p>

<p>squash</p>

<p>fast-forward</p>

<p>reflog</p>
</div>


!SLIDE 
# Git Core Concepts #

!SLIDE center
git is a DAG (directed acyclic graph)
![DAG](img/dag.png)

!SLIDE 
DAG nodes each represent a commit object

!SLIDE 
A commit object's SHA is derived from it's contents

<pre>
$ git cat-file -p 561c944                         
tree a5223bcfed6ed6c30c480c5467a9c09e6d87ba45
parent 1dbdf0f43c1e90b4396ca2dd56054458dc45a590
author Ted Naleid <contact@naleid.com> 1328481167 -0600
committer Ted Naleid <contact@naleid.com> 1328481167 -0600

my special commit message
</pre>

!SLIDE 
commits are completely immutable and are _impossible_ to accidentally destroy with git commands

<div class="smallercentered">
you can still <code>rm -rf yourrepo</code> and lose anything not yet pushed out
</div>

<div class="smallestcentered">
you also can easily destroy uncommitted work, so commit early & often
</div>

!SLIDE 

!SLIDE 

!SLIDE 
you cannot modify commits, only add new ones

!SLIDE 
garbage collection is the only truly destructive action in git

!SLIDE 
garbage collection will not destroy anything if something is pointing to it

!SLIDE 
four things point at nodes:
- child nodes
- tags 
- branches 
- the reflog

!SLIDE 
child nodes point an N parent nodes (normally 1, or 2 for a merge)

!SLIDE 
tags are non-floating node pointers

!SLIDE 
branches are floating node pointers that you can "reset" to move wherever you want

- "remote" branches are just more pointers in your local repo that point to a node 
- there is nothing really remote about them, they are the last known commits that a remote repo was pointing at
- you can see all the branches under .git/refs/heads (local) and .git/refs/remotes (remote)
- a branch is just a pointer to a commit
- commits don't "belong to" branches, they're just commits that a branch might point at, but the branch pointer is easily moved
- the rest of the branch is implied by the ancestry of that commit
- unlike other source control systems, there's no metadata in the commit that says it's part of a branch

!SLIDE 
the reflog is a log of recent branch movement, by default it contains two weeks of history

!SLIDE 
the reflog is unique to an instance of a repo, even if something has been gc in one repo it could still exist in another

!SLIDE 
if nothing is pointing at a node, it's "dangling" (or "detached")

!SLIDE 
it is subject to garbage collection, but still exists till garbage collection removes it

!SLIDE 
git does a gc when the number of "dangling" objects hits a threshold, something like every 1000 commits 

!SLIDE 
you can have the courage to experiment as you can always get back


!SLIDE 
git reset [--soft|--hard] _SHA_
- points HEAD & the current branch to the specified SHA (stop if --soft)
- clean index, make it look like SHA (stop if default/--mixed)
- clean working copy, make it look like SHA (--hard)

(from: http://progit.org/2011/07/11/reset.html)


!SLIDE 
git reset --hard HEAD - means clean out the working directory and any staged information, don't move the branch pointer

(git amend example)

("delete"/recover commits example)


!SLIDE 
git reset --soft SHA

useful for easily squashing commits down into a single commit


!SLIDE 
# what is rebasing #

To reapply a series of changes from a branch to a different base, and reset the head of that branch to the result.

!SLIDE 
private activity, should't be done on things that have been pushed publicly 

!SLIDE 
Assume the following history exists and the current branch is "topic":

<pre>
                         A---B---C topic
                        /
                    D---E---F---G master
</pre>

From this point, the result of either of the following commands:

   git rebase master
   git rebase master topic

would be:

<pre>
                         A'--B'--C' topic
                        /
           D---E---F---G master
</pre>

!SLIDE 
# what is squashing #

<pre>
git merge --squash
</pre>



!SLIDE 
# what is fast forwarding #



!SLIDE 
# what is cherry picking #


!SLIDE 
# what is fetch vs pull #

Fetching a branch means to get the branch's head ref from a remote repository, to find out which objects are missing from the local object database, and to get them, too. 



!SLIDE 
the right way to pull down changes from the server

   - stash
   - fetch - retrieve objects


git smart-pull is the git version of hg's fetch, git fetch+stash+(rebase -p || ff)+unstash…nice docs too
   


!SLIDE 
# Viewing tools #

   tower
   prompt with current branch & SHA
   git status --short
   omglog
   git l (graph log)
   git ld (graph with decorators only)
   gbrt
   unreachable commits
