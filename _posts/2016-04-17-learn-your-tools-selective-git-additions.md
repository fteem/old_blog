---
layout: post
title: "Learn Your Tools: Selective Git Additions"
tags: [git, add, patch]
image: selective-git-additions.png
---

In my career, if I ever get asked to interview someone, I would always start the 
interview with version control. Maybe it's a bit of an exaggeration, but to me, 
understanding version control and having a good grasp of programming logic is 
more important than just knowing the syntax and the stdlib of a programming 
language. 

But, for whatever reason, whether it's laziness or simply conformism, we often 
learn a tool until a certain point that works for us and we stop. We are happy 
with what we know at that point and we rarely open the documentation to see how 
we can improve our skills. In this blog post let's tackle a Git command that we 
all know and use - `git add`.

## `git add`

This is for the ones that do not know (or just don't remember) what `git add` 
does. From the documentation, available at your nearest terminal by executing
`man git-add`:

```
This command updates the index using the current content found in the working tree, to prepare the content staged for the next commit. It typically adds the current content of
existing paths as a whole, but with some options it can also be used to add content with only part of the changes made to the working tree files applied, or remove paths that
do not exist in the working tree anymore.
```

What this says is that when you run `git add` on a path, it "prepares" this path
to be commited in the next commit. You can read the rest of the documentation on
`git add` online or in your terminal.

So, an example, right? Well, this is pretty easy. Let's create a new repo using
the `git init` command. Create a new directory, called `patching`:

```bash
mkdir patching && cd patching
```

After that, you can run the `git init` command which will initialize a new blank
Git repo:

```bash
➜  patching git init
Initialized empty Git repository in ~/projects/patching/.git/
```

Great, now we have a new repo that we can play with. Let's create a plain text
(`touch text_file`) file whose contents will be the first paragraph of the 
"lorem ipsum" dummy text:

```
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor 
incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis 
nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. 
Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu 
fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in 
culpa qui officia deserunt mollit anim id est laborum.
```

Let's check our repo now, by executing `git status`:

```bash
➜  patching git:(master) ✗ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	text_file

nothing added to commit but untracked files present (use "git add" to track)
```

As you can see, we have the `text_file` file as an untracked file. Let's use
`git add` to track it and commit it after:

```
➜  patching git:(master) ✗ git add text_file
➜  patching git:(master) ✗ git commit -m 'new text file'
[master (root-commit) fdebf0b] new text file
 1 file changed, 1 insertion(+)
 create mode 100644 text_file
```

Great, that's the usual flow of using `git add` and `git commit` in combination.
But, as I am sure you know, Git is quite more powerfull than this. Let's see
how we can be selective about our the changes to our files with the help of
`git add`.

## Patching

Let's introduce couple of changes to our `text_file`. We are going to make all
leters upcased on the second line, all leters lowercased on the fourth line.
Also, we are going to delete the last line of the file. When we do a `git diff`
on the file, we should get something like this:

```diff
diff --git a/text_file b/text_file
index 7b95d6b..0585259 100644
--- a/text_file
+++ b/text_file
@@ -1,6 +1,5 @@
 Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor
-incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis
+INCIDIDUNT UT LABORE ET DOLORE MAGNA ALIQUA. UT ENIM AD MINIM VENIAM, QUIS
 nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.
-Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu
+duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu
 fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
-culpa qui officia deserunt mollit anim id est laborum.
```

Great! Now, take this for example. As a part of our next commit, we would like
to only commit the deletion of the last line, but not the other edits of our 
file. Basically, our commit should have only the changes (deletions) on the last 
line of the file, but nothing else.

How would you do this? One option is to revert all the changes to the file, and
then just remove the last line. Sure, that's one way. But what if the file had
very complex changes that were hard to revert and redo again?

Git has this beautiful flag on the `git add` command - `-p` or `--patch`. Let's
check it's documentation:

```
-p, --patch
    Interactively choose hunks of patch between the index and the work tree and add them to the index. This gives the user a chance to review the difference before adding
    modified contents to the index.
```

If we run `git add -p`, this command will take us over all of the changes that
occured in the files that are tracked in the repo and ask us for an action. 
Let's give it a shot:

```diff
➜  patching git:(master) ✗ git add -p

diff --git a/text_file b/text_file
index 7b95d6b..0585259 100644
--- a/text_file
+++ b/text_file
@@ -1,6 +1,5 @@
 Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor
-incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis
+INCIDIDUNT UT LABORE ET DOLORE MAGNA ALIQUA. UT ENIM AD MINIM VENIAM, QUIS
 nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.
-Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu
+duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu
 fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
-culpa qui officia deserunt mollit anim id est laborum.
Stage this hunk [y,n,q,a,d,/,s,e,?]?
````

Let's analyse the output quickly. It is composed of two things - the diff and the
prompt by Git. The diff represents all of the changes that happened to the file.
The interesting part here is the prompt by Git. Instead of supplying any of the
options, let's enter a question mark and press return:

```
Stage this hunk [y,n,q,a,d,/,s,e,?]? ?
y - stage this hunk
n - do not stage this hunk
q - quit; do not stage this hunk or any of the remaining ones
a - stage this hunk and all later hunks in the file
d - do not stage this hunk or any of the later hunks in the file
g - select a hunk to go to
/ - search for a hunk matching the given regex
j - leave this hunk undecided, see next undecided hunk
J - leave this hunk undecided, see next hunk
k - leave this hunk undecided, see previous undecided hunk
K - leave this hunk undecided, see previous hunk
s - split the current hunk into smaller hunks
e - manually edit the current hunk
? - print help
```

As you can see, Git will show us all of the available options for this prompt
with their respective descriptions. We could go at length about all of these
options and how they are useful, but for the purpose of this tutorial we will
stick to our original usecase.

Since now Git threats all of the changes as a single hunk, we need to separate
changes into different hunks. We can do that by supplying the `s` argument,
which will do that for us:

```diff
Stage this hunk [y,n,q,a,d,/,s,e,?]? s
Split into 3 hunks.
@@ -1,3 +1,3 @@
 Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor
-incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis
+INCIDIDUNT UT LABORE ET DOLORE MAGNA ALIQUA. UT ENIM AD MINIM VENIAM, QUIS
 nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.
```

As you can notice, after supplying the `s` argument, Git informed us that the
changes were split into three hunks. Now, we can go over each of the changes and
add them selectively. All we have to do now is to enter `y` for the changes we
need (the first and the last change), and enter `n` for the second change:

```diff
Stage this hunk [y,n,q,a,d,/,s,e,?]? s
Split into 3 hunks.
@@ -1,3 +1,3 @@
 Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor
-incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis
+INCIDIDUNT UT LABORE ET DOLORE MAGNA ALIQUA. UT ENIM AD MINIM VENIAM, QUIS
 nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.
Stage this hunk [y,n,q,a,d,/,j,J,g,e,?]? y
@@ -3,3 +3,3 @@
 nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.
-Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu
+duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu
 fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
Stage this hunk [y,n,q,a,d,/,K,j,J,g,e,?]? n
@@ -5,2 +5 @@
 fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
-culpa qui officia deserunt mollit anim id est laborum.
Stage this hunk [y,n,q,a,d,/,K,g,e,?]? y
```

Now, if we execute `git status`, we will see the following output:

```
➜  patching git:(master) ✗ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   text_file

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   text_file
```

As expected, Git staged some of our changes that we said that we want. We can 
see them by executing `git diff --cached`:

```diff
diff --git a/text_file b/text_file
index 7b95d6b..233552e 100644
--- a/text_file
+++ b/text_file
@@ -1,6 +1,5 @@
 Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor
-incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis
+INCIDIDUNT UT LABORE ET DOLORE MAGNA ALIQUA. UT ENIM AD MINIM VENIAM, QUIS
 nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.
 Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu
 fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
-culpa qui officia deserunt mollit anim id est laborum.
```

There we go - only the first and the last change of the file were added to our
staging area. Now we can freely go and commit these stagges, while the second
change is still unstaged in our repo.

## The power is yet to be unleashed

The blog post started as a bit of a rant of how we don't learn our tools. Sure,
we are all guilty of it, including me. This is the main reason that got me to 
write this post.

We have so much power at our fingertips that we need to utilize. And the only way
that we can do it is by RTFM and a lot of practice. Or, the other way. If you
are doing a lot of practice (i.e. a lot of work), always think about how you can
improve your speed and execution. If you knew how powerful are some of the tools
that you are using, your speed and effectiveness would increase exponentionally.

If we only took the time to read the documentation...

