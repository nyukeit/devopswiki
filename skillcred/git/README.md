## Basic Commands

### Configure

Configure the local git client with variables.

```bash
git config --global user.name "your_name"
```

```bash
git config --global user.email "you@example.com"
```

View your custom configuration

```bash
git config --list
```

### Initialize

Initialize an empty repository in a directory. This does not make Git start tracking your files.

```bash
git init
```

### Branching

Working with Git is essentially working with commit branches. They help keep code clean and disasters away.

```bash
git branch main #Creates a branch called 'main'
```

> We have moved away from using 'master' as a branch name since it is biased nomenclature. You can use it if you want, but we will all judge you.

```bash
# View all branches in the repo
git branch
```

> The little start next to the branch name means the HEAD is pointing to that branch as of the current commit.

### Clone

Clone a remote repository (eg. hosted on GitHub) into a local working directory. This will also create a folder with the same name as the repository.

```bash
git clone <https://example.com/user/repo.git
```

### Track Files

When initializing a new repository, you need to specify the files that Git should track.

```bash
git add <filename> # Track a particular file
git add . # Track all files in the current folder
git add -a # Same as above
```

### Check Status of Local Repository

This shows you the current status of your local repository and if there modifications or not.

```bash
git status
```

### Commit Changes

This is essentially like saving a file. Once the files are committed, they are ready to be pushed. Commits are done with a message and summary that highlights the modifications made.

```bash
git commit -m "Your commit message here"
```

-still-developing-