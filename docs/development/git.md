# Git Guide

## Naming Conventions

We use naming conventions for branches and commits to automatically connect them to JIRA tickets
(see [GitHub for Jira](https://www.atlassian.com/blog/jira-software/github-for-jira)).

### Branch naming

We name branches according to the following scheme: `<type>/<short-description>[-<JIRA-KEYS>|optional]`.

Where _type_ can be one of "fix", "feat" (_feature_), "task" (e.g. for Rake tasks), "docs" or "chore" (for everything else).

You can optionally add the corresponding JIRA ticket ID as a suffix
(this way you can automatically connect the branch to the ticket).

This suffix is `jira_link` Git `prepare-commit-msg` hook to suggest a ticket number to add
to the commit description (see [Git hooks](./lefthook.md)).

Examples: `fix/profile-avatar-PROJECT-42`, `feature/families-PROJECT-11`, `chore/docker-dev`.

### Commits naming

Commit message should be of the form: `"<type>: <desc> <'[ci skip]'|optional>"`.

Where _type_ can be one of "fix", "feat", "task" (e.g. for Rake tasks), "docs", "ci", "danger" (for Danger rules) or "chore" (for everything else).

Please, use short and descriptive messages.

Add the link to the corresponding issue in the commit description, e.g.:

```sh
# The second -m will add the text to the commit description
$ git commit -m "chore: add docker-compose.yml" -m "https://hicommon.atlassian.net/browse/PROJECT-35"
```

That would make links to the issues clickable on GitHub (just click on `...` near the commit name)
and even locally, if you're using a _link-aware_ terminal (e.g. VS Code):

```sh
$ git show -s --format=%B 6994e09
chore: add docker-compose.yml

https://hicommon.atlassian.net/browse/PROJECT-35
```

**TIP**: add Git alias for showing commit descriptions:

```sh
git config --global alias.msg "show -s --format=%B"
git msg 6994e09
```

Use "[ci skip]" to prevent CI builds if commit doesn't contain anything _testable_: `"[ci skip] docs: update readme"`.

Use commit message _body_ to provide more context to the change. That would help others in the future when they'll try to _blame_ you)
