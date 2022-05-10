# Some simple rebase exercises

## Exercise 1
We have 1 branch, and master has moved forward. When you only have 1 branch, it's basically always safe to run `git rebase master`.
```
git checkout ex1b1
git rebase master
git log
```

## Exercise 2
Here we have `ex2b1` based off master, and `ex2b2` based off `ex2b1`. They both rebase cleanly.
```
git checkout ex2b1
git rebase master
git checkout ex2b2
git rebase ex2b1
git log
```

## Exercise 3
Here we have `ex3b1` based off master, and `ex3b2` based off of an older commit of `ex3b1`. They both rebase cleanly, git is smart enough to pick up the new commit from `ex3b1`.
```
git checkout ex3b1
git rebase master
git checkout ex3b2
git rebase ex3b1
git log
```

## Exercise 4
Here we have 2 `ex4b1` based off master, and `ex4b2` based off of a commit of `ex4b1` that no longer exists. This can happen if `ex4b1` had a commit ammended or `ex4b1` got rebased and had to have conflicts resolved. When you try to rebase `ex4b2` onto `ex4b1`, you'll notice you have your favorite: MERGE CONFLICTS! Yay. But why are there conflicts? `ex4b2` didn't modify `a`.

Choose the top change in the conflict, and finish the rebase. You'll see that the log looks like what you might want it to look like
```
git checkout ex4b1
git rebase master
git checkout ex4b2
git rebase ex4b1

<choose top value>

git add -u
git rebase --continue
git log
```

## Exercise 5
Same as ex4, except we're going to choose the bottom change!

Choose the bottom change in the conflict, and finish the rebase. You'll see that there are now 2 duplicated looking commits with the same messages, and the second one actually undos some changes. This rebase has done 2 unintended things: made extra commits AND undone changes. Bad. Your teammates and yourself will not be happy.
```
git checkout ex5b1
git rebase master
git checkout ex5b2
git rebase ex5b1

<choose bottom value>

git add -u
git rebase --continue
git log
```

## Exercise 6
Same as ex4, except we're going to rebase a smarter way.

Use `git rebase --onto <dest base commit> <everything AFTER this commit>` to rebase in a more explicit way. This works by copying every commit after the specified one onto the destination commit.
```
git checkout ex6b1
git rebase master
git checkout ex6b2
git rebase ex6b1
git rebase --onto ex6b1 d1b7ed4b7a36f56620cf2084354573ca076aee47
git log
```

# Why is using --onto "necessary"?
It's not necessary, but it can save a lot of manual work and confusion along the way. Things like duplicated commits, accidentally undone commits, and resolving the same conflict over and over.

If you understand what the `--onto` flag does in ex6, then think of it this way:
```
git rebase branchName
```
is equivalent to
```
git rebase --onto branchName <first commit in this branch that is on master>
```
which means: take every commit that has occurred since this split from master, and move then onto `branchName`. But why since the split from master, why not since I branched off on `ex*b1`? Well, becuase git keeps track of things with SHAs, not branch names. When you branch from something, you don't tell git to branch from another branch, or tag, you tell git to branch from a SHA. A branch name is just a pointer in time to a SHA. So when `ex*b1` moves, `ex*b2` doesn't know that, it just points to whatever SHA `ex*b1` pointed to at the time that you created `ex*b2`.

So when you have a branch-on-a-branch like `ex*b2`, there are commits between master and that branch that were actually part of `ex*b1`. So wouldn't those commits be duplicated? They could have been, but git is smart enough to detect duplicate commits that happen in the same order. So when you do `git checkout ex2b2; git rebase ex2b1`, git sees that the content of the commit in SHA `beb622`  (not the sha itself, the content. The sha of the first commit in `ex2b1` after the rebase is `fe5912`) already exists so it will skip it. Thats why both `ex2` and `ex3` work well.

When it comes to `ex[4-6]` we ran into some sort of problem. Why? Because the content of the commits on `b2` where NOT THE SAME as the commits on `b1`, they had been changed. So during `git rebase ex4b1`, it compares the content of `ex4b1` `b0553f` to `f72bf4` and they are different, hence the merge conflict.

Now, the future path depends on how you resolve the merge conflict. If you resolve it so that the content looks the same as in `b0553f`, then git will notice that it's a duplicate commit and skip it, then continue on with the next commit. This is what we did in `ex4`. If you resolve it so that the content does NOT look the same, like in `ex5`, then git sees that the commits are NOT the same, and adds a new commit, same commit message but the content of `a` goes back to how it was on `ex4b2`, undoing the ammended commit on `ex4b1` but leaving it in the history.

This is why I *always* use `--onto` when I have a branch on a branch, we skip all this trouble of comparing commits. We know more than git knows, so we tell it to 100% skip all commits until an explicit one. No conflict resoltion necessary :magic:.
