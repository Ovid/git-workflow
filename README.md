# git-workflow

This repo contains a simplified subset of the git tools used by [All Around
the World](https://allaroundtheworld.fr/) for our software development. It
makes it dead easy for teams using git (and in our case, github) to work
together.

There are only three new commands to remember:

* `git hub $issue_number` (create new branch based on a github ticket)
* `git refresh` (rebase your work on top of current `master`)
* `git merge-with-master` (cleanly add your branch back to `master`)

Note: only the `bin/git-hub` command assumes you're using github. The other
commands work fine without it.

# Assumptions

The `master` branch is never worked on directly. Instead, new branches are
created (usually for an individual github ticket), we hack, we regularly pull
new changes into that branch, and after a pull request is created and the
approved, we merge the code back into `master`.

The examples below assume the files in the `bin/` directory are in your path.
If they are not in yoru path, you have to type the commands explicitly:

    bin/git-hub 5738
    bin/git-refresh
    bin/git-merge-with-master

## Checking Out a Branch

We assume that branches are _usually_ per ticket. You can manually create a
branch, but we tend not to. Instead, if you have been assigned (or taken)
github issue 5738, with the title "Rework reputation to handle faction
conflict", you run the following command:

    git hub 5738

And you're in a shiny new branch named `rework-reputation-to-handle-faction-conflict-5738`.

If that branch already exists, it's checked out.

### Caveats

1. The new branch that is created is based on the branch you're on, so you
   probably want to run this from `master`
2. Branch detection is based on a heuristic, using the ticket number. If you
   have more than one branch with that ticket number, you will get a warning
   and the code will exit.
3. You'll need a config file for this. See `perldoc bin/git-hub` for this.
4. Assumes Perl 5 version 20
5. The following non-standard modules must be installed from the CPAN:

* `autodie`
* `Config::General`
* `IPC::System::Simple`
* `JSON`
* `Text::Unidecode`
* `Pithub`

## Refreshing Your Branch

It's often the case that you want to pull in the latest `master` to keep your
code up-to-date. Working on a change for a week with a fast-moving codebase
can cause serious headaches when it's time to merge. Thus, you should regularly
run the following in your branch (I try to run it at least once per day):

    git refresh

Regardless of the branch you are on, this code:

* Stashes changes (if any)
* Checks out master
* Does a fast-forward merge
* Checks out your branch (if branch is not master)
* Rebases onto `master` (if branch is not master)
* Pops changes from stash, if any

In other words, it cleanly rebases your code on top of master, even if you
have uncommitted changes.

## Merging into `master`

For many companies, a typical git history graph looks like a plate of
spaghetti someone's just unswallowed onto your monitor. This makes it very
hard to dig through your git history, to follow changes, or use tools like
`git-bisect`. For us, we simply run:

    git merge-with-master

And that will cleanly update `master` and rebase your branch on top of it, and
push that change to your origin.

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
