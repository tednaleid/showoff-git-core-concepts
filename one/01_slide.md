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
# four things point at commits #

child commits

tags 

branches 

the reflog

!SLIDE 
# child commits #

point at 1..N parent nodes 

<pre>
                              E---F---G 
                             /
                        A---B---C---D 
</pre>



!SLIDE 
# tags #
fixed node pointers

<pre>
                        A---B---C 
                                ↑
                           release_1.0
</pre>

<pre>
$ git commit -m "adding stuff to C"
</pre>

<pre>
                        A---B---C---D 
                                ↑
                           release_1.0
</pre>


!SLIDE 
# branches #

floating node pointers that move as you add commits

<pre>
                        A---B
                            ↑
                          master
</pre>

<pre>
$ git commit -m "adding stuff to B"
</pre>

<pre>
                        A---B---C
                                ↑
                              master
</pre>


!SLIDE 
# branches #

they're just pointers, and are easy to move if you don't like where they are at
<pre>
                        A---B---C
                                ↑
                              master
</pre>

<pre>
$ git reset --hard SHA_OF_B
</pre>

<pre>
                        A---B---C
                            ↑
                          master
</pre>

<div class="smallercentered">
commit <code>C</code> still exists and was not harmed by moving the pointer
</div>
<div class="smallestcentered">
we'll talk more about <code>reset</code> in a bit
</div>

!SLIDE 
# remote branches #
"remote" branches are just pointers in your local repo

<pre>
                              origin/master
                                    ↓
                    A---B---C---D---E
                            ↑       
                          master    
</pre>

<div class="smallercentered">
for most commands, there's nothing remote about them, they're just moved on a <code>fetch</code> or <code>pull</code>
</div>

!SLIDE 
# branches #
branches are in <code>.git/refs/heads</code> (local) and <code>.git/refs/remotes</code> (remote)

<pre>
$ ls -1 .git/refs/heads/**/*
.git/refs/heads/master
.git/refs/heads/my_feature_branch
</pre>

<pre>
ls -1 .git/refs/remotes/**/*  
.git/refs/remotes/origin/HEAD
.git/refs/remotes/origin/master
.git/refs/remotes/origin/my_feature_branch
</pre>

!SLIDE 
# branches #

All a branch object file contains is the SHA of the commit it's pointing at

<pre>
$ cat .git/refs/heads/master 
0981e8c8ffbd3a1277dda1173fb6f5cbf4750d51
</pre>

<pre>
$ git cat-file -p 0981e8c8ffbd3a1277dda1173fb6f5cbf4750d51
tree 4fd7894316b4659ef3f53426166697858d51a291
parent e324971ecf1e0f626d4ba8b0adfc22465091c100
parent d33700dde6d38b051ba240ee97d685afdaf07515
author Ted Naleid &lt;contact@naleid.com&gt; 1328567163 -0800
committer Ted Naleid &lt;contact@naleid.com&gt; 1328567163 -0800

merge commit of two branches
</pre>

!SLIDE 
# branches #
commits don't "belong to" branches, unlike other source control systems, there's nothing in the commit metadata about branches it belongs to

!SLIDE 
# branches #
a branch's commits are implied by the ancestry of the commit the branch points to

<pre>
                                   feature
                                      ↓
                              E---F---G 
                             /
                        A---B---C---D 
                                    ↑ 
                                  master
</pre>


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
public rebasing is bad because it creates partially redundant commit states and can't get to your node from theirs

example

!SLIDE
if you want to clean things up, the alternative is to create another branch, rebase onto that and push it out


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

!SLIDE 
tower

   cmd-click branch to see whole tree (similar to omglog)

   prompt with current branch & SHA
   git status --short
   omglog
   git l (graph log)
   git ld (graph with decorators only)
   gbrt
   unreachable commits
