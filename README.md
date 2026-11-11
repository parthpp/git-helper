Utilities that helps for better commit messages. 
Commit subject [should not be more than 80 characters](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html). 
But Jira ticket numbers and PR numbers takes about ~15-20 character of this valuable real estate.   
These utilities help with moving them to [git trailers](https://git-scm.com/docs/git-interpret-trailers).

## Hooks

[Hooks](https://git-scm.com/docs/githooks) are scripts git runs by
itself at set moments; you never call them.

### 1. Commit: `prepare-commit-msg`

Runs on every `git commit`, before the editor opens (also with `-m`).
Adds the Jira ticket from your branch name to every commit message. Name
your branch so it ends with `/PROJ-<number>`. Start name with what makes
it unique, so tab completion finds it after a few letters instead of after typing `PROJ-` every time.

```
branch:  login-retry/PROJ-123
commit:  Fix login retry

         Jira: PROJ-123
```

If the branch has no ticket, it asks once:

```
Jira ticket (PROJ-123, Enter to skip):
```

Not every change has a ticket, so just press Enter and the commit goes
ahead without one.

### 2. Push to main: `pre-push`

Runs on every `git push`, before anything is sent. When pushing to
`main`/`master`, it lists commits that have no Jira trailer or a line
over 80 characters (URLs excepted). Then it asks:

```
warning: these commits have no Jira trailer:
  a1b2c3d Fix login retry
warning: these commit messages break the line rules:
  e4f5a6b Retry the login request once when the access token expires in the middle of a session: subject is 85 chars (max 80)
Push anyway? [y/n]
```

This is useful for commits that directly go to `master`

## Merge a PR: `git squash-merge`

Webstorm and other local editors provide far superior commit writing experience than Github UI, especially for squash merges.  
`git-squash-merge` is a git command. It works as a git command because git runs
any program named `git-<name>` on your PATH as `git <name>`. It
squash-merges the PR for your current branch, with a message you write
in your editor. The draft is: the PR title, the PR's commits as a bullet
list that should be edited, and
[git trailers](https://git-scm.com/docs/git-interpret-trailers):

```
Fix login retry after token expiry

* Retry once on 401

* Add test for expired token

Pull-Request: https://github.com/acme/app/pull/42
Jira: PROJ-123
Signed-off-by: Jane Doe <jane@acme.com>
```

After you save, any warnings are
shown with the final message:

```
----- final commit message -----
Fix login retry after token expiry

When the access token expired mid-session, the first request failed
with a 401 and the user was logged out. Now we refresh the token and
retry the request once before giving up.

Pull-Request: https://github.com/acme/app/pull/42
Signed-off-by: Jane Doe <jane@acme.com>
--------------------------------
warning: no 'Jira: PROJ-123' trailer
Squash-merge PR #42 into main? [y]es/[e]dit/[N]o:
```

Where the message lives: in a temp file while you edit. Nothing is committed
locally. `gh` sends the subject and body to GitHub, which creates the
squash commit on `master`. If you abort or the merge fails, your message is
kept in `.git/squash-merge-msg-<PR>` and offered next time. Your branches
are left as they are.

## Install and configure

Clone this repo anywhere; `~/git-helper` is used below.

**Hooks, for every repo:**

```sh
git config --global core.hooksPath ~/git-helper/hooks
```

**Command** A git alias or can be symlinked to PATH as an executable

```sh
git config --global alias.squash-merge '!~/git-helper/git-squash-merge'
```

Set Jira project key used by these scripts as a git config like

```sh
git config --global githelper.ticketPrefix BLUE
```

To use WebStorm on git commands and make git wait until you close
the tab:

```sh
git config --global core.editor "webstorm --wait"
```
