# Silvermine Coding Standards: Commit History

All Silvermine codebases use git for version control. This gives us phenomenal
powers to review the history of the codebase and see where and why a change was
introduced. To get the most of that power, we have strict rules about our
commit history, outlined in this document.

A commit history can easily be polluted when commits are not well-crafted. The
basic principle is **each commit should be for a single piece of work, well
described, and not contain unrelated changes**.

Here are some rules about commits:

**Commits that contain only code review changes are never allowed.** Let's say
that you start a branch for a new feature or a bugfix, and that branch contains
one or more commits. Then you get your work reviewed and the reviewer requests
changes. You *must rebase* and fix the actual commit that introduced the
problem the reviewer wants you to fix. For example, if he wants you to rename a
variable or fix a whitespace error, go fix it in the commit that you introduced
the error (or poorly-named variable) in.

Why is this so important? Later on, some poor developer is going to be trying
to trace the history of a piece of code to see when a change was introduced.
Imagine if they do git blame or git log and look at the commits that introduced
a change, but that change was just to fix whitespace or rename a variable. This
is very frustrating. Or, if the code review comment was more substantial - an
actual refactoring of the work you did - then the work that should be contained
in one autonomous commit is broken into two or more commits. This makes it very
hard to retrace the steps that actually led to a piece of code being
introduced.

*Remember that this "poor developer" that we speak of could be you*. Be kind to
your future self, or to those who succeed you.

**Commits should be for a single piece of work.** Do not make one commit do
many things. For example, "fixed foo and added new bar functionality" is not a
good commit. Future you will come back to see where a piece of code was added,
and coming upon this commit as the place it originated will now have to figure
out "was the code added to fix foo? or to add bar?"


## Rebasing

As you can tell from the comments above, knowledge of how to do a git rebase is
very important to your workflow. You will be expected to use a rebase to fix
code that is in a branch you're requesting to have merged. Thus, become
proficient at rebasing if you are not already. If you need help, [Pro
Git](https://git-scm.com/book/en/v2/) has a good [chapter on
rebasing](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History)

**IMPORTANT:** when you rebase for a pull request, *do not rebase off of master
unless there are merge conflicts*. If you rebase off master between the first
review and the re-review, the reviewer now has to re-review the entire pull
request rather than just what you changed since he last reviewed the code. *You
should only rebase off of master when it is absolutely necessary due to merge
conflicts, and even then you should work with the reviewer to know when he
would like that done. Perhaps it could be done after the review is 100%
complete.*

Instead of rebasing off master, you want to rebase off the last common commit
that you have with master.

For example, imagine this scenario where you created a branch off of master and
while you were working on it, several new commits were added to master.

```
Master:

A   B   C   D   E   F   G   H
*---*---*---*---*---*---*---*
         \
Your      \---*---*---*---*
Branch        R   S   T   U
```

You've added commits R, S, T, and U to your branch after branching off of
master's commit C. Thus, your commit history (`git log`) is A, B, C, R, S, T,
U.

At this point if you rebased off of master (e.g. `git rebase origin/master`),
you would end up with this history:

```
Master:

A   B   C   D   E   F   G   H
*---*---*---*---*---*---*---*
                             \
Your                          \---*---*---*---*
Branch                            R2  S2  T2  U2
```

That means your history is now A, B, C, D, E, F, G, H, R2, S2, T2, U2. (R
through U have "2" added because they are now new commits that are different
from the original R through U). It also means that when your reviewer compares
the difference in commits U and U2 (your original last commit and your new last
commit) they will also be seeing all of the changes that you pulled in from
master (namely, D through H). This makes the re-review painful and typically
leads to the reviewer not being able to review just your changes, but requires
them to re-review everything in your branch (R2 through U2).

The better solution is to rebase off of your common commit (C in this example),
which will give you a history like this:

```
Master:

A   B   C   D   E   F   G   H
*---*---*---*---*---*---*---*
         \
Your      \---*---*---*---*
Branch        R2  S2  T2  U2
```

In this scenario, each of your commits changed, but their common ancestor (C)
did not. Thus, `git diff U..U2` will allow the reviewer to see only the changes
that you introduced since he reviewed it last. **This will make not only the
reviewer's life better, but also your own** because it means your code can be
merged sooner.


### Rebasing Off Common Ancestor

Given all that we talked about above, you may have the question **how do I
rebase off the common ancestor of my feature branch?**

This is a two-step process:

```
# Find the common ancestory between your branch and the branch you are trying to
# merge into (in this case the merge target is master):
MERGE_BASE=$(git merge-base master my_branch)

# Then do your interactive rebase off of that commit:
git rebase -i ${MERGE_BASE}
```


## Commit Messages

We care a great deal about having good commit messages. Chris Beams wrote a
good blog post about why good commit messages matter. Please read and apply his
post: http://chris.beams.io/posts/git-commit/

Additional notes:

   * We should put **“Refs #xxxxx”** (where xxxxx is the issue number) in the
     subject all commit messages, preferably as the first thing in the subject.
      * The article shows it at the end, but git log --oneline shows only the
        subject, and it’s really handy to have the issue number there.
      * In some cases a commit references multiple issues. In that case, you
        can list those issues at the end of the body of the commit message and leave it
        out of the subject.
   * We don’t have a hard-and-fast 50 char rule for the subject, but should
     really try to keep it under 69 characters.
   * Use markdown formatting in your commit messages, including a space before
     either an asterisk or a numbered list if doing ordered/unordered lists. This
     formats them properly in most source control and project management systems.
   * Commit messages should focus more on explaining the "why" than the how.
     The commit itself (the code changes) represent the "how". Someone will be
     looking at the git history (your message) to figure out *why* you did what you
     did.

Here is an example of a commit we might craft. It explains what and why more
than how. The how is shown by seeing the code changes with `git show $hash`.

```
Refs #12345 Remove flux capacitor from DeLorean

The annual corporate meeting will occur on October 3, 2015 this year,
and big announcements will be made at that time. However, Marty McFly
and Doc Brown will both have traveled forward to October 21, 2015, and
in the pages of *Gray's Sports Almanac* will be copies of the
announcements made at the annual meeting. When they return to 1985, they
will unfortunately turn traitor and spread the word of the future
announcements, which we need to prevent.

There were several ways we could have prevented this from happening:

 1. Intercept the lightning in front of courthouse on November 12, 1955 at precisely 10:04pm, trapping both Marty and the Doc in 1955
 2. Destroy the partial bridge over the Shonash/Clayton/Eastwood Ravine prior to September 4, 1885
 3. Remove the flux capacitor from the DeLorean before Marty and the Doc could take their first trip on October 21, 1985

I chose option three because I was actually alive at the time. If I'd
chosen options one or two it would have required that I travel back to
1955, which in itself could have caused problems with the time-space
continuum. It was also hoped that by completely removing the flux
capacitor it would prevent Marty and the Doc from traveling to any other
time periods and spoiling other great advances in our organization's
future. Of course, the Doc is known to be resourceful and has even used
[steam](https://c4.staticflickr.com/8/7416/10344460816_4d0ce7cfdf_b.jpg)
to power at least one flux capacitor, so even this fix may not be
sufficient.

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# Your branch is ahead of 'origin/master' by 2 commits.
#   (use "git push" to publish your local commits)
#
# Changes to be committed:
#       modified:   src/tests/lib/TimeSpaceContinuum.js
#       modified:   src/tests/vehicles/DeLorean.test.js
#       modified:   src/vehicles/DeLorean.js
#
```
