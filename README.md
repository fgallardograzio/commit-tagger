# commit-tagger

## Description

commit-tagger is a collection of shell scripts and git hooks meant to help developers consistently prefix each commit message with the appropriate tags. These are automatically generated based on the current branch name and the current branch *parents*.

Popular branch naming conventions suggest prefixing branches with certain strings according to the branch type, such as `feature/`, `hotfix/`, `bugfix/`, etc. For this reason, only what's after the `/` will be taken into account by commit-tagger.

For example, by default, a commit on branch `develop` will have its message automatically prefixed with `[develop]`, and a commit on branch `feature/KEY-1234` will have its message prefixed with `[KEY-1234]`.

This is particularly handy when the branch name contains, for example, a JIRA issue identifier.

When checking out a new<sup>[1](#branch-creation)</sup> branch, a hierarchical relationship between the base branch and the new branch is automatically established, and commits on the new branch will contain tags for both the current and the parent branch.

These relationships can also be explored and managed using the commands `get-parent`, `set-parent`, `remove-parent`, `ignore`, `unignore` and `is-ignored`.

This relationship model maintains certain correlation with the way that JIRA handles sub-tasks, but it's not limited to only two branches. This feature can be used to automatically include tags from up to 20 *ancestors*.

commit-tagger also allows developers to run a series of commands before actually issuing the commit. If any of those commands fail (exit with a status other than 0), the commit will be aborted. Any unstaged changes will be temporarily stashed before running those commands, so that only what's about to be commited is affected and taken into account. Immediately after this step is completed the working directory is restored.

This can be used to check file names and contents, checking code for syntax errors, code style, running unit tests, and everything a developer needs to make sure what's being commited is perfect.

Please be careful not to interrupt this process by pressing `Ctrl+C` **more than once**<sup>[2](#interrupting-the-pre-commit-hook)</sup>.

## Installation

Installation consists of two independent steps.

The first step is installing commit-tagger on your system. To accomplish this you're welcome to use the provided `install` script.

Make sure `install` is executable.

<pre>chmod +x install</pre>

Then, install commit-tagger by running:

<pre>./install &lt;installation-path&gt;</pre>

If `<installation-path>` is not provided, it will be installed in `~/bin`. If the provided path does not exist, it will be automatically created after prompting the user for confirmation.

You can also do this step manually by placing `git-commit-tagger` in any directory in your PATH, and copying `hooks` and `files` with all of their contents into `~/.commit-tagger`.

Note that if you choose to do this step manually you won't be able to use `uninstall` later on.

**Please make sure that the directory in which commit-tagger is installed is in your system PATH.**


The second step is actually installing commit-tagger's hooks in a repository.

You can do this by running:

<pre>git commit-tagger install-hooks</pre>

## Usage

Usage: git commit-tagger &lt;command&gt; [&lt;args&gt;]

### Available commands

* tag [&lt;branch&gt;]
* get-parent [&lt;branch&gt;]
* set-parent [&lt;branch&gt;] &lt;parent&gt;
* remove-parent [&lt;branch&gt;]
* ignore [&lt;branch&gt; [&lt;branch&gt; ...]]
* unignore [&lt;branch&gt; [&lt;branch&gt; ...]]
* is-ignored [&lt;branch&gt;]
* install-hooks
* uninstall-hooks
* version
* help

### Commands description

#### tag

Get the full list of tags for &lt;branch&gt;, or the current branch if omitted.

#### get-parent

Get the name of the parent of &lt;branch&gt;, or the current branch if omitted.

#### set-parent

Specify &lt;parent&gt; as the parent branch of &lt;branch&gt;, or the current branch if omitted.
The &lt;parent&gt; branch's tag will be included in the commit messages of &lt;branch&gt;.

#### remove-parent

Remove the parent associated to &lt;branch&gt;, or the current branch if omitted.

#### ignore

This will result in &lt;branch&gt;, or the current branch if omitted, being ignored.
Branches created with &lt;branch&gt; as base won't automatically have &lt;branch&gt; set as parent.

#### unignore

No longer ignore &lt;branch&gt;, or the current branch if omitted.

#### is-ignored

Shows whether or not &lt;branch&gt;, or the current branch if omitted, is being ignored.

#### install-hooks

Install commit-tagger's hooks into the current git repository.

#### uninstall-hooks

Remove commit-tagger's hooks from the current git repository.

#### version

Output version message.

#### help

Show help and usage.


## Configuration

Usage: git config commit-tagger.&lt;setting&gt; &lt;value&gt;

### Available settings

#### hooksEnabled

Allowed values: `true` and `false`. Default: `true`.

Enable all commit-tagger's hooks in the current repository.

#### checkoutEnabled

Allowed values: `true` and `false`. Default: `true`.

Enable the 'post-checkout' hook.

#### commitMsgEnabled

Allowed values: `true` and `false`. Default: `true`.

Enable the 'prepare-commit-msg' hook to tag messages.

#### preCommitEnabled

Allowed values: `true` and `false`. Default: `true`.

Enable the 'pre-commit' hook to run custom commands.

#### preCommitVerbose

Allowed values: `true` and `false`. Default: `true`.

Show the output of all the tests.

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
