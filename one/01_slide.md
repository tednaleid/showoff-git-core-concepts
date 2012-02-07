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

!SLIDE 
git is a DAG (directed acyclic graph)

<pre>
                      E---F---G 
                     /
                A---B---C---D-----------K---L---M 
                             \         /
                              H---I---J
                                           
</pre>


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
garbage collection is the only truly destructive git action 

!SLIDE 
garbage collection will not destroy a commit if something is pointing to it

!SLIDE 
# four things point at commits #

child commits

tags 

branches 

the reflog

!SLIDE 
# child commits #

point at 1..N parent commits 

<pre>
                              E---F---G 
                             /
                        A---B---C---D 
</pre>



!SLIDE 
# tags #
fixed commit pointers

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

floating commit pointers that move as you add commits

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
commits don't "belong to" branches, there's nothing in the commit metadata about branches

!SLIDE 
# branches #
a branch's commits are implied by the ancestry of the commit the branch points at

<pre>
                                   feature
                                      ↓
                              E---F---G 
                             /
                        A---B---C---D 
                                    ↑ 
                                  master
</pre>

<div class="smallercentered">
<code>master</code> is <code>A..D</code> and <code>feature</code> is <code>A,B,E..G</code>
</div>

!SLIDE
# HEAD #

<code>HEAD</code> is the currently active commit and will be a parent of the next commit
<pre>
$ cat .git/HEAD
ref: refs/heads/master
</pre>

<div class="smallercentered">
most of the time it points to a branch, but can point directly to a SHA when "detatched"
</div>


!SLIDE 
# reflog #
a log of recent <code>HEAD</code> movement

<pre>
$ git reflog                                       
d72efc4 HEAD@{0}: commit: adding bar.txt
6435f38 HEAD@{1}: commit (initial): adding foo.txt
</pre>

<pre>
$ git commit -m "adding baz.txt"
</pre>

<pre>
$ git reflog                                       
b5416cb HEAD@{0}: commit: adding baz.txt
d72efc4 HEAD@{1}: commit: adding bar.txt
6435f38 HEAD@{2}: commit (initial): adding foo.txt
</pre>


<div class="smallercentered">
by default it contains up to two weeks of history
</div>

!SLIDE 
# reflog #
the reflog is unique to a repo instance

if a commit has been garbage collected it could still exist in a clone

!SLIDE 
# dangling commit #
if nothing is pointing at a commit, it's "dangling"

!SLIDE 
# dangling commit #
<pre>
                    A---B---C---D---E---F
                                        ↑
                                      master
</pre>

<pre>
$ git reset --hard SHA_OF_B
</pre>

<pre>
                    A---B---C---D---E---F
                        ↑
                      master

</pre>

<div class="smallercentered">
<code>C..F</code> are now dangling
</div>

!SLIDE 
# dangling commit #

but they will be safe for ~2 weeks because of the reflog

<pre>
                                     HEAD@{1}
                                        ↓
                    A---B---C---D---E---F
                        ↑
                      master

</pre>

<div class="smallestcentered">
<code>HEAD@{1}</code> will become <code>HEAD@{2}</code>...<code>HEAD@{N}</code> as <code>HEAD</code> moves on in the reflog
</div>

!SLIDE 
# garbage collection #
once a dangling commit leaves the reflog, it is at risk of garbage collection

!SLIDE 
# garbage collection #
git does a <code>gc</code> when the number of "dangling" objects hits a threshold

<div class="smallestcentered">
something like every 1000 commits 
</div>

!SLIDE 
# garbage collection #
to prevent garbage collection, just point something at it

<pre>
$ git tag mytag SHA_OF_C
</pre>

!SLIDE 
you should have courage to experiment 

you can always get back to prior commits if something doesn't work


!SLIDE
# Git Command Tips #


!SLIDE 
# reset --soft #
<pre>
git reset --soft &lt;SHA&gt;
</pre>

1. moves <code>HEAD</code> & the current branch to the specified <code>&lt;SHA&gt;</code>
2. index - unchanged
3. working directory - unchanged

<p/>
<div class="smallestcentered">
useful for easily squashing commits down into a single commit
</div>

!SLIDE 
# reset (default)#
<pre>
git reset [--mixed] &lt;SHA&gt;
</pre>

1. moves <code>HEAD</code> & the current branch to the specified <code>&lt;SHA&gt;</code> 
2. clean the index, make it look like <code>&lt;SHA&gt;</code> 
3. working directory - unchanged

<p/>
<div class="smallestcentered">
<code>git reset HEAD</code> will unstage everything in the index
</div>

!SLIDE 
# reset --hard#
<pre>
git reset --hard &lt;SHA&gt;
</pre>

1. moves <code>HEAD</code> & the current branch to the specified <code>&lt;SHA&gt;</code> 
2. clean the index, make it look like <code>&lt;SHA&gt;</code> 
3. clean the working copy, make it look like <code>&lt;SHA&gt;</code> 

<p/>
<div class="smallestcentered">
dangerous if you have uncommitted changes, useful for undoing bad commits
</div>
<div class="smallestcentered">
for more info, see: http://progit.org/2011/07/11/reset.html
</div>

!SLIDE 
# reset --hard HEAD #
<pre>
git reset --hard HEAD
</pre>

<div class="smallercentered">
just means clean out the working directory and any staged information, don't move the branch pointer
</div>

!SLIDE
(git amend example)

("delete"/recover commits example)

!SLIDE 
# rebasing #

rebasing reapplies a series of changes to another parent commit

it then resets the head of that branch to the result

!SLIDE 
# rebasing #

<pre>
                                feature+HEAD
                                      ↓
                              E---F---G 
                             /
                        A---B---C---D 
                                    ↑ 
                                  master
</pre>

<pre>
   git rebase master
</pre>

<pre>
                        (dangling but still in reflog)
                                      ↓
                              E---F---G
                             /
                        A---B---C---D---E'--F'--G'
                                    ↑           ↑ 
                                  master  feature+HEAD
</pre>

!SLIDE 
# rebasing #
a private activity, should't be done on things that have been pushed publicly 

!SLIDE
# rebasing #
public rebasing is bad because it creates redundant commits with different SHAs 

git also has trouble moving remote branch pointers to follow a rebase

!SLIDE
# rebasing #
if you want to clean things up, an alternative is to create another branch, rebase onto that and push it out

!SLIDE 
# squashing #

Compresses unmerged commits down into a single commit and appends to the destination branch

!SLIDE 
# squashing #

<pre>
                                           feature
                                              ↓
                                      E---F---G 
                                     /
                        A---B---C---D 
                                    ↑ 
                               master+HEAD
</pre>

<pre>
git merge --squash feature
</pre>

<pre>
                                           feature
                                              ↓
                                      E---F---G 
                                     /
                        A---B---C---D---G' 
                                        ↑ 
                                   master+HEAD
</pre>

<div class="smallestcentered">
squashing is used to clean up history, when the thinking behind <code>E..F</code> is unimportant
</div>


!SLIDE 
# cherry picking #

Apply the changes from an individual commit to the current branch

<pre>
                              E---F---G 
                             /
                        A---B---C---D 
                                    ↑ 
                               master+HEAD
</pre>

<pre>
git cherry-pick SHA_OF_F
</pre>

<pre>
                              E---F---G 
                             /
                        A---B---C---D---F' 
                                        ↑ 
                                   master+HEAD
</pre>


!SLIDE 
# fetch #

Fetching means to get the current branch's position in the remote repository, plus any missing commits, but don't move local references

!SLIDE 
# fetch #

<pre>
                     A---B---E---F   
(origin)                         ↑ 
                              master (local ref in remote repo)  


                   origin/master
                         ↓
(local)              A---B---C---D 
                                 ↑ 
                              master+HEAD
</pre>

<pre>
$ git fetch
</pre>


<pre>
                         origin/master
                               ↓
                           E---F
                          /
(local)              A---B---C---D 
                                 ↑ 
                              master+HEAD
</pre>


!SLIDE 
# pull #

<code>pull</code> is <code>fetch</code> plus <code>merge</code>. It gets remote refs and commits and merges with your current head.

!SLIDE 
# pull #

<pre>
                     A---B---E---F   
(origin)                         ↑ 
                              master (local ref in remote repo)  


                   origin/master
                         ↓
(local)              A---B---C---D 
                                 ↑ 
                              master+HEAD
</pre>

<pre>
$ git pull
</pre>


<pre>
                         origin/master
                               ↓
                           E---F----
                          /         \
(local)              A---B---C---D---G 
                                     ↑ 
                                  master+HEAD
</pre>

!SLIDE 
the "right" way to pull down changes from the server

1. <code>stash</code> any uncommitted changes (if any)
2. <code>fetch</code> the latest refs and commits from origin
3. <code>rebase -p</code> your changes (if any) onto origin's head
4. else, just fast-forward your head to match origin's
5. un-<code>stash</code> any previously stashed changes

<p>
<div class="smallercentered">
<code>fetch</code> + <code>rebase</code> avoids unnecessary commits
</div>

!SLIDE
Luckily, <code>git smart-pull</code> (part of the git-smart ruby gem) does all this for us

<pre>
gem install git-smart
</pre>


!SLIDE 
# Useful Git Tools #

!SLIDE center
# Tower.app #

<div class="smallestcentered">
tip: cmd-click a branch to unselect it and see the whole tree
</div>

!SLIDE
# Terminal #
decorate your prompt with current branch & SHA

ex: https://bitbucket.org/tednaleid/shared-zshrc/src/tip/zshrc_prompt

!SLIDE
# git aliases #
   
<pre>
[alias]
  # nice one liner for status
  st = status --short    


  # remove files from index
  unstage = reset HEAD
</pre>

<div class="smallestcentered">
put these in your <code>~/.gitconfig</code>
</div>


!SLIDE
# git aliases #
   
<pre>
[alias]
  # pretty ascii graph log format
  l = log --graph --pretty='%Cred%h%Creset -%C(yellow)%d%Creset\
          %s %Cblue[%an]%Creset %Cgreen(%cr)%Creset'\
          --abbrev-commit --date=relative


  # pretty log with all branches
  la = !git l --all


  # show just commits currently decorated by branch/tag pointers 
  # really useful for high level picture
  ld = !git l --all --simplify-by-decoration 
</pre>

<div class="smallestcentered">
put these in your <code>~/.gitconfig</code>
</div>


!SLIDE
# git aliases #
   
<pre>
[alias]
  # all commits unreachable via branch, tag, or child commit
  # ignores the reflog 
  # so it displays all commits in jeopardy of garbage collection
  lost-commits = !"for SHA in $(git fsck --unreachable\ 
                 --no-reflogs | grep commit |\
                 cut -d\\  -f 3); do git log -n 1 $SHA; done"
</pre>

<div class="smallestcentered">
put these in your <code>~/.gitconfig</code>
</div>

!SLIDE
# omglog #

OSX only…watches file system for changes &amp; auto updates ascii graph log

<pre>
gem install omglog
</pre>

<div class="smallestcentered">
currently requires ruby 1.9.X
</div>


!SLIDE
# Questions? #

