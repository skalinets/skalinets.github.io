---
title: Automatically Append Ticket IDs to Git Commit Messages with Pre-Commit
---

If your workflow involves including ticket IDs in branch names and commit messages, automating this process can save time and help maintain consistency. Here’s how to set up a pre-commit hook that automatically appends a ticket ID to the end of every commit message line, assuming your ticket IDs follow the pattern DS-XXX.

# Prerequisites:
- Git
- [pre-commit](https://pre-commit.com/) installed (If you don’t have it, install via `pip install pre-commit`, or `brew isntall pre-commit` or visit their site for detailed instructions).

# Create the Hook Script
In your repository, create a new directory to store your scripts, if you haven't already. Within this directory, create a new file named `add-issue-id-to-commit-msg.sh`.

``` bash
#!/bin/sh

# Extract the current branch name
BRANCH_NAME=$(git symbolic-ref --short HEAD)

# Define the regex pattern for the issue ID
ISSUE_ID_REGEX="DS-[0-9]+"

# Check if the branch name contains the issue ID
if [[ $BRANCH_NAME =~ $ISSUE_ID_REGEX ]]; then
    ISSUE_ID=${BASH_REMATCH[0]}
    # Append the issue ID to the last line of the commit message
    sed -i.bak -e "s/$/ [${ISSUE_ID}]/" "$1"
fi
```

Make the script executable:

``` bash
chmod +x scripts/add-issue-id-to-commit-msg.sh
```

## Update Pre-Commit Configuration

Edit your `.pre-commit-config.yaml` file to include the script as a hook:

```yaml
repos:
  - repo: local
    hooks:
      - id: append-issue-id
        name: Append issue ID to commit message
        entry: scripts/add-issue-id-to-commit-msg.sh
        language: script
        stages: [commit-msg]
```

## Install the Pre-Commit Hook
Ensure pre-commit is configured to run the commit-msg hook:


``` bash
pre-commit install -t commit-msg
```

## Test the Hook

Create a branch with a name containing a ticket ID (e.g., feature/add-login-DS-1234) and make a commit. The pre-commit hook will automatically append the ticket ID DS-1234 to the end of your commit message.

## Conclusion

This setup ensures that your commits are consistently tagged with ticket IDs, improving traceability across your project. As a bonus, this approach requires minimal effort from your development team, reducing the risk of human error.
