
This repo contains a simplified subset of the git tools used by [All Around
the World](https://allaroundtheworld.fr/) for our software development. It
makes it dead easy for teams using git (and in our case, github) to work
together.

There are only three new commands to remember:

* `git refresh` (rebase your work on top of the current default branch)
* `git pushback` (pushes your changes to origin)
* `git done` (cleanly add your branch back to the default branch)

There is an optional `git-hub` command available:

* `git hub $issue_number` (optional. create new branch based on a github ticket)

Only the `bin/git-hub` command assumes you're using github. The other commands
work fine without it.

# Assumptions

The default branch (usually `main` or `master`) is never worked on directly.

We derive the default branch via:

    basename $( git symbolic-ref refs/remotes/origin/HEAD )

If you get the error, `ref refs/remotes/origin/HEAD is not a symbolic ref`,
you can fix it with this:

     git symbolic-ref refs/remotes/origin/HEAD refs/remotes/origin/$branch_name

Where \$branch_name is the name of the primary branch you develop from
(`main`, `master`, etc.).

Instead, new branches are created (usually for an individual ticket or change
request), we hack, we regularly pull new changes into that branch, and after a
pull request is created and the approved, we merge the code back into the
default branch.

The examples below assume the files in the `bin/` directory are in your path.
If they are not in yoru path, you have to type the commands explicitly:

    bin/git-hub 5738   # don't use if you don't use github
    bin/git-refresh
    bin/git-pushback
    bin/git-done

# The Commands

## Checking Out a Branch

We recommend all work being done on a new branch that is not the "primary"
branch. Thus, from your primary branch:

    git checkout -b my-shiny-new-branch

Optionally, if you are using github, we assume that branches are _usually_ per
ticket. You can manually create a branch, but we tend not to. Instead, if you
have been assigned (or taken) github issue 5738, with the title "Rework
reputation to handle faction conflict", you run the following command:

    git hub 5738

And you're in a shiny new branch named `rework-reputation-to-handle-faction-conflict-5738`.

If that branch already exists, it's checked out.

This command is not required. Instead, create your own version if you need it.
If you do want it, this command, see [Config File](#config-file) below.

## Refreshing Your Branch

It's often the case that you want to pull in the latest the default branch to
keep your code up-to-date. Working on a change for a week with a fast-moving
codebase can cause serious headaches when it's time to merge. Thus, you should
regularly run the following in your branch (I try to run it at least once per
day):

    git refresh

Regardless of the branch you are on, this code:

* Checks out master
* Does a fast-forward merge
* Checks out your branch (if branch is not master)
* Rebases onto the default branch (if branch is not already the default branch)

In other words, it cleanly rebases your code on top of the target branch.

If you have uncommitted changes, `git refresh` will print an error an exit. If
you pass `--stash` (or `-s`), it will stash your changes, run the commands,
and the pop your changes. However, this means that if you have a rebase
conflict, you will have stashed changes you will need to remember to pop off
the stash.

## Pushing back to origin

For many workflows, you will need to create a pull request (or merge request,
if you're using gitlab). The easiest way to ensure that `origin` always has
the latest changes is to use:

    git pushback

That only works on branches. It's equivalent to:

    git push --set-upstream origin your-branch-name

If you must force the push, use `--force`. Internally, we use
`--force-with-leaase` which will overwrite the remote changes, but only if
no one else has pushed changes there.

    git pushback -f
    # or
    git pushback --force

## Merging into the default branch

For many companies, a typical git history graph looks like a plate of
spaghetti someone's just unswallowed onto your monitor. This makes it very
hard to dig through your git history, to follow changes, or use tools like
`git-bisect`. For us, we simply run:

    git done

And that will cleanly update the default branch and rebase your branch on top
of it, and push that change to your origin.

With that, you get a clean git history like this:

    | * 44eba1b094 - ...
    | * 217350810f - ...
    |/
    *   c84e694e59 - Merge branch 'no-add-message-from-context-object-8615' PR#8622 (6 days ago)  <some author>
    |\
    | * 9d73143f75 - ...
    | * 983a1a5020 - ...
    | * e799ecc8e3 - ...
    | * aa9c981c2e - ...
    | * 4651830fd6 - ...
    |/
    *   010e94b446 - Merge branch 'fix-test-combat-module-usage' PR#8623 (7 days ago)  <some author>
    |\
    | * 46d8917af7 - ...
    |/
    *   4acfdd8309 - Merge branch 'economy-action-use-args-hashref' PR#8617 (10 days ago)  <some author>
    |\
    | * a1f863f908 - ...
    | * b3dc3efb2a - ...
    | * ab77373fca - ...
    | * b5491e4ae9 - ...

And when you're done, it will also print out the instructions on how you can
delete your local and remote copies of the branch, to help you be nice and not
leave branches cluttering up the repository.

# Config File

If you use the `git hub` command, it requires that the current directory have
a `.git-hub` file in the following format:

    user   Ovid
    repo   git-workflow
    token  <personal access token>

See the [github documentation for creating a personal access
token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token).

We strongly recommend that you get a "read-only" token for use with this. If
it had more privileges, falling into the wrong hands might be a security hole.

If you do not use the `bin/git-hub` tool, you do not need a config file.

# Caveats

These caveat's only apply to `bin/git-hub`.

1. The new branch that is created is based on the branch you're on, so you
   probably want to run this from the default branch
2. Branch detection is based on a heuristic, using the ticket number. If you
   have more than one branch with that ticket number, you will get a warning
   and the code will exit.
3. You'll need a config file for this. See `perldoc bin/git-hub` for this.
   There is a `git-hub-config-example` file included that you can rename to
   `.git-hub`.
4. Assumes Perl 5 version 8
5. If the ticket title has non-ASCII characters, you must have
   `Text::Unidecode` installed with Perl.
6. [jq](https://stedolan.github.io/jq/) must be installed if you use the `git-hub` script
   (https://stedolan.github.io/jq/)

# Changes

2021-10-28 

    - `git-refresh` no longer automaticallys stashes uncommitted
      changes. It was a great feature, but it turns out it's an
      anti-feature for those new to `git`. If there was a rebase
      conflict, they'd lose their changes because they would forget
      to run `git stash pop` after.

    - We now automatically detect the main branch you should be
      branching off of, refreshing from, and pushing back to. 
      Only `bin/git-hub` requires a config file now.

2021-10-24

    - `git done` used to be called `git merge-with-master`, but with so
      many projects having a different "source" branch name,
      `merge-with-master` does not make sense any more. `git done`
      behaves exactly the same as `merge-with-master`, but uses the
      correct target branch.

# ACKNOWLEDGEMENTS

The following have added several valuable suggestions to this project.

* Peter Mottram
* Noel Maddy
* Branislav Zahradník
