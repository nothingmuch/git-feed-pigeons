# Senility Management with/for Git

This tool tries to make automatic snapshotting of the working directory easy.

## Snapshotting

    git breadcrumb [ --trail=foo [<object>] ]

This will return `refs/breadcrumbs/HEAD@{0}`, which is might have been
updated:

If there are no changes to the working directory (`git stash-create` returns
nothing), then `refs/breadcrumbs/HEAD` is updated to the `HEAD` commit.

Otherwise, the working directory has changes captured in a new stash commit.
Store that in `refs/breadcrumbs/HEAD`, unless the trees of that commit and both
its parents match those those of the tip of `refs/breadcrumbs`.

If `HEAD` is a symbolic ref, then `refs/breadcrumbs/%s` is updated along with
it (much like `git reflog` entries are saved).

If `--trail` is passed, `refs/breadcrumbs/trails/%s` is also updated too.

The reflogs for these refs will be much denser than those of a normal branch,
with an entry for potentially every directory tree change, not for every
commit

`git stash apply refs/breadcrumbs/foo@{0}` may also be used on breadcrumb refs
in the same manner.

If an object is specified (presumably the output of a previous breadcrumb
command), `--trail`. This is just like `git update-ref` but with a set prefix,
and ensures that a reflog is started.

### TODO

- [ ] `--trail` option - already in opt spec
  - [ ] explicit object argument

## Tagging

Breadcrumbs should be annotated with notes to make them useful. For example,
test outputs.

    git notes --ref=namespace -m foo

    ref="fail"; go test 2>&1 test.out && ref="pass"; git notes --ref=tests.$ref -F test.out $breadcrumb

### TODO

- [ ] Utilities for exit code to suffix, namespace from args, filtering, etc.
- [ ] Utility for --amending arbitrary trees to note commits. LFS support? tag a commit?

## Hooks, Automating

### TODO

- [ ] git hooks, crontab (?), watcher, editor, build tools, tests
- [ ] generic wrapper for CLI patterns to build notes?
- [ ] env vars, hostname etc in namespace
- [ ] go test handling
  - [ ] go test
  - [ ] go test -run XyZ
  - [ ] go test -v
  - [ ] go test -race
  - [ ] go test -cover
  - [ ] go test -short
  - [ ] go test -bench, benchmem (host? load)
  - [ ] go test -trace
  - [ ] profiling
  - [ ] go build
  - [ ] gofmt
  - [ ] go metalinter
- [ ] skip goddamn false negatives
    
## Reflog filtering

### TODO

- [ ] git log --reflog --first-parent refs/breadcrumbs/HEAD
  - [ ] notes filtering
  - [ ] can we get it in gitg?
- [ ] tail -f for git reflog (filter with peco)
- [ ] log <-> reflog conversion (or just viewing?)
  - using grafts + filter branch? maybe just funny args to git log?
- [ ] filter by notes http://stackoverflow.com/questions/13118127/filter-git-log-to-show-only-commits-with-notes

# Background

As I bravely venture into programming with diminishing mental faculties, I am
finding it harder to keep track of the intended state of my working directory.

Let me put it another way, if I find treasures in git stash, I must be doing it
wrong.

Obviously the solution is to store even more data named using cryptic numbers,
and to abstract it away with another layer of concepts!

## What happened to my files?

There are two modes in which I'm changing the state of my working directory.

The first is as a result of git operation of some kind, I checkout, rebase,
merge etc and the magic happens and all is well. Git keeps a handy log of my
mistakes in its `reflog`.

The second is when I'm out trying various combinations of symbols, hunting for
little islands of coherent logic in software. Fortunately I don't have to
actually understand what's going on, that's what tests are for.

I'm meant to quantize that work using the tools, i.e. if I've reached firm
ground I try to remember to commit it. If it's still shaky, I might stash it.
But if I'm trying to build an island out of floating trash and debris, how do I
hoarde all my findings?

## What? Who is git stash?

Back in the day we used to create git stash entries by hand. Both ways!

    ## 1. git stash create

    # commit the state of the index
    index_commit=$( echo "index on master: $orig_head foo" | \
    	git commit-tree -p HEAD $(git write-tree) )

    # add the unstaged changes
    git add -u

    # the WIP commit is a bit funny, it merges
    # the index commit and HEAD
    wip_commit=$( echo "WIP on master: $orig foo" | \
       git commit-tree -p HEAD -p $index_commit $(git write-tree) )

    ## 2. remaining steps of git stash save

    # update the stash ref:
    touch "$(git rev-parse --git-path logs/refs/stash)" # reflog is necessary!
    git update-ref -m "update the damn stash!" refs/stash $wip_commit

    # finally reset the working directory
    git reset --hard

The result of this can be applied with `git stash apply`. The two separate
commits give git everything it needs to recreate the index and the working
directory.

## Notes?

Notes are just a parallel branch (one per namespace), that contains files whose
names are commit IDs and whose contents are the notes' bodies.

Since these notes are versioned too, it often makes sense to just overwrite the
previous note.

Breadcrumb notes should be treated as positive evidence, not as states, i.e.
tests passing and tests failing are both evidence of events occurring, and
should be saved in different namespaces to avoid confusion. Idempotent
breadcrumb notes are better suited for later analysis.

### TODO

- [ ] explain how to embed arbitrary artifacts in notes
- [ ] http://who-t.blogspot.co.uk/2015/07/using-git-notes-for-marking-test-suite.html
- [ ] https://github.com/swipely/jenkins-git-notes-plugin
