# commit-tagger

## Description

commit-tagger is a collection of shell scripts and git hooks meant to allow developers to:

1. Consistently prefix every commit message with one or more `[tags]`. These are automatically generated based on the current branch name and the current branch *parents*.

2. Run a series of user-defined commands before actually issuing a commit.

### Tagging commit messages

Popular branch naming conventions suggest prefixing branches with certain strings according to the branch type, such as `feature/`, `hotfix/`, `bugfix/`, etc. For this reason, only what's after a `/` will be taken into account by commit-tagger.

For example, by default, a commit on branch `develop` will have its message automatically prefixed with `[develop]`, and a commit on branch `feature/KEY-1234` will have its message prefixed with `[KEY-1234]`.

This is particularly handy when the branch name contains, for example, a JIRA issue identifier.

Commits that are result of a merge, squash, amend, template or ones that have a reused message won't be affected. Only `git commit`, `git commit -m` and `git commit -F` will trigger this functionality.

When tagMode is set to `opt-out` (default behavior) it's possible to prevent commit-tagger tags from being automatically added by prefixing the commit's message with a `-` (minus sign / hyphen).

A tagged message can be manually generated for any branch using the command `tag`. The output of this command is used by the prepare-commit-msg hook.

When checking out a new<sup>[1](#branch-creation)</sup> branch, a hierarchical relationship between the base branch and the new branch is automatically established, and commits on the new branch will contain tags for both the current and the parent branch.

These relationships can also be explored and managed using the commands `get-parent`, `set-parent`, `remove-parent`, `show-parents`, `ignore`, `unignore`, `is-ignored` and `show-ignored`.

This relationship model maintains certain correlation with the way that JIRA handles sub-tasks, but it's not limited to only two branches. This feature can be used to automatically include tags from up to 20 *ancestors*.

The following table shows some example commands and the generated commit message. Note that a space will be added between the last tag and the message, unless it begins with a `[`.

| Commands (on branch: task/TAG-123)                                   | Commit message                                 |
| -------------------------------------------------------------------- | ---------------------------------------------- |
| git commit -m "Some message"                                         | \[TAG-123\] Some message                       |
| git commit -m "\[Another-Tag\] With a custom tag"                    | \[TAG-123\]\[Another-Tag\] With a custom tag   |
| git checkout -b subtask/TAG-456<br>git commit -m "With parent's tag" | \[TAG-123\]\[TAG-456\] With parent's tag       |
| git commit -m "-No tags in this message"                             | No tags in this message                        |

### Running pre-commit commands

commit-tagger also allows developers to run a series of commands before actually issuing a commit. If any of those commands fail (exit with a status other than 0), the commit will be aborted. Any unstaged changes will be temporarily stashed before running those commands, so that only what's about to be commited is affected and taken into account. Immediately after this step is completed the working directory is restored.

This can be used to check file names and contents, checking code for syntax errors, code style, running unit tests, and everything a developer needs to make sure what's being commited is perfect.

This hook's execution can be prevented by using git commit's option `--no-verify` (`git commit --no-verify`).

Please be careful not to interrupt this process by pressing `Ctrl+C` **more than once**<sup>[2](#interrupting-the-pre-commit-hook)</sup>.

To configure custom pre-commit tasks use the command `set-precommit`. This will open the current repository's pre-commit hook's configuration file in git's core.editor (`git config core.editor`) or vi if undefined.

Note that pre-commit commands won't be run during a merge.

## Installation

Installation consists of two independent steps.

The first step is installing commit-tagger on your system. To accomplish this you're welcome to use the provided `install` script.

Make sure `install` is executable.

<pre>chmod +x install</pre>

Then, install commit-tagger by running:

<pre>./install &lt;installation-path&gt;</pre>

If `<installation-path>` is not provided, commit-tagger will be installed in `~/bin`. If the provided path does not exist, it will be automatically created after prompting the user for confirmation.

You can also do this step manually by placing `git-commit-tagger` in any directory in your PATH, and copying `hooks` and `files` with all of their contents into `~/.commit-tagger`.

Note that if you choose to do this step manually you won't be able to use `uninstall` later on.

**Please make sure that the directory in which commit-tagger is installed is in your system PATH.**

The second step is actually installing commit-tagger's hooks in a repository.

You can do this using the command `install-hooks`. Run:

<pre>git commit-tagger install-hooks</pre>

## Usage

Usage: git commit-tagger &lt;command&gt; [&lt;args&gt;]

### Available commands

* tag [&lt;branch&gt;] &lt;message&gt;
* get-parent [&lt;branch&gt;]
* set-parent [&lt;branch&gt;] &lt;parent&gt;
* remove-parent [&lt;branch&gt;]
* show-parents
* ignore [&lt;branch&gt; [&lt;branch&gt; ...]]
* unignore [&lt;branch&gt; [&lt;branch&gt; ...]]
* is-ignored [&lt;branch&gt;]
* show-ignored
* set-precommit
* install-hooks
* uninstall-hooks
* clean
* version
* help

### Commands description

#### tag

Usage: git commit-tagger tag [&lt;branch&gt;] &lt;message&gt;

Prefix &lt;message&gt; with the full list of tags for &lt;branch&gt;, or the current branch if omitted.

#### get-parent

Usage: git commit-tagger get-parent [&lt;branch&gt;]

Get the name of the parent of &lt;branch&gt;, or the current branch if omitted.

#### set-parent

Usage: git commit-tagger [&lt;branch&gt;] &lt;parent&gt;

Specify &lt;parent&gt; as the parent branch of &lt;branch&gt;, or the current branch if omitted. The &lt;parent&gt; branch's tag will be included in the commit messages of &lt;branch&gt;.

#### remove-parent

Usage: git commit-tagger remove-parent [&lt;branch&gt;]

Remove the parent associated to &lt;branch&gt;, or the current branch if omitted.

#### show-parents

Usage: git commit-tagger show-parents

List all the branches and their parents, separated by a colon, one per line.

#### ignore

Usage: git commit-tagger [&lt;branch&gt; [&lt;branch&gt; ...]]

This will result in &lt;branch&gt;, or the current branch if omitted, being ignored. Branches created with &lt;branch&gt; as base won't automatically have &lt;branch&gt; set as parent.

#### unignore

Usage: git commit-tagger unignore [&lt;branch&gt; [&lt;branch&gt; ...]]

No longer ignore &lt;branch&gt;, or the current branch if omitted.

#### is-ignored

Usage: git commit-tagger [&lt;branch&gt;]

Shows whether or not &lt;branch&gt;, or the current branch if omitted, is being ignored.

#### show-ignored

Usage: git commit-tagger show-ignored

List all the branches commit-tagger is currently ignoring, one per line.

#### set-precommit

Usage: git commit-tagger set-precommit

Open in your editor the file containing the commands to be run by the pre-commit hook.

#### install-hooks

Usage: git commit-tagger install-hooks

Install commit-tagger's hooks into the current git repository.

#### uninstall-hooks

Usage: git commit-tagger uninstall-hooks

Remove commit-tagger's hooks from the current git repository.

#### clean

Usage: git commit-tagger clean

Remove all data associated to commit-tagger, such as branches' parents, ignored branches and custom pre-commit commands.

#### version

Usage: git commit-tagger version

Output version message.

#### help

Usage: git commit-tagger help

Show help message and usage.


## Configuration

Usage: git config commit-tagger.&lt;setting&gt; &lt;value&gt;

Example: git config commit-tagger.hooksEnabled false

### Available settings

#### hooksEnabled

Allowed values: `true` and `false`. Default: `true`.

Enable all commit-tagger's hooks in the current repository.

#### checkoutEnabled

Allowed values: `true` and `false`. Default: `true`.

Enable the 'post-checkout' hook to automatically set the branch's *parent* after checking out a new branch.

#### commitMsgEnabled

Allowed values: `true` and `false`. Default: `true`.

Enable the 'prepare-commit-msg' hook to tag new commit's messages automatically.

#### preCommitEnabled

Allowed values: `true` and `false`. Default: `true`.

Enable the 'pre-commit' hook to run custom commands before actually issuing the commit.

#### preCommitVerbose

Allowed values: `true` and `false`. Default: `true`.

Show the output of the commands run by the pre-commit hook.

#### tagIgnored

Allowed values: `true` and `false`. Default: `true`.

Tag commit messages on ignored branches.

#### confirm

Allowed values: `true` and `false`. Default: `true`.

Ask for confirmation before actually issuing the commit. Only when -m or -F is used.

#### tagMode

Allowed values: `opt-in` and `opt-out`. Default: `opt-out`.

opt-in: Do not tag commit messages unless prefixed with a "+".

opt-out: Tag all commit messages automatically unless prefixed with a "-".


## Uninstalling commit-tagger

Just like for the installation, uninstalling commit-tagger involves different processes.

The first one is removing all of commit-tagger's hooks from a given repository. You can use the command `uninstall-hooks` to quickly do this. Run:

<pre>git commit-tagger uninstall-hooks</pre>

You may also want to remove all data associated to commit-tagger in a given repository. This includes the list of branches' parents, the list of ignored branches and all custom pre-commit commands.

You can do this by running:

<pre>git commit-tagger clean</pre>

Alternatively, you can manually delete `.git/commit-tagger.parents`, `.git/commit-tagger.ignored` and `.git/commit-tagger.precommit` from a repository.

A different process is actually removing commit-tagger from your system. This involves deleting git-commit-tagger from the installation directory, and removing `~/.commit-tagger` and all of its contents.

In order to do this easily you can use the provided `uninstall` script, if you used `install` in the first place. Make sure the file has execution rights and then run:

<pre>./uninstall</pre>

The list of files and folders that will be removed is presented, and you will be prompted for confirmation.


## Known issues and limitations

### Branch creation

Since branches in git are nothing more than pointers to a specific commit, there's no point in distinguishing a newly created branch and an old branch if both reference the same commit.

For this reason, git provides no way to determine if a branch has been just created *after* it's been checked out.

So, the post-checkout hook sets the previous branch as the parent of the current branch only when both point to the same commit, none of them are being ignored, and no parent is already defined for the current branch.

### Interrupting the pre-commit hook

Due to the way that git runs hooks, the shell does not appear to actually wait for a hook's interrupt trap to finish executing.

That's why aborting a commit by pressing `Ctrl+C` while pre-commit commands are being executed will result in them being interrupted immediately, and restoring the working directory from stash is done *asynchronously* in the background.

Please be careful since this process can also be interrupted by pressing `Ctrl+C` again, which will leave your working directory empty or missing unstaged files.

To fix this, you'll have to check the stash list and manually run:

<pre>
git reset --hard HEAD
git stash pop --index
</pre>

