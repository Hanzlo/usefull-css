# Gitflow Workflow

FER+ projects uses the [Gitflow branching model](http://nvie.com/posts/a-successful-git-branching-model):

- the **master** branch contains the latest **stable** version
- the **develop** branch contains the latest **unstable** development version
- all stable versions are tagged using semantic versioning

Gitflow is really just an abstract idea of a Git workflow. This means it dictates what kind of branches to set up and how to merge them together.

- [Develop and Master Branches](#develop-and-master-branches)
- [Feature Branches](#feature-branches)
- [Release Branches](#release-branches)
- [Hotfix Branches](#hotfix-branches)

## Develop and Master Branches

Instead of a single master branch, this workflow uses two branches to record the history of the project. The master branch stores the official release history, and the develop branch serves as an integration branch for features. It's also convenient to tag all commits in the master branch with a version number.

```bash
git branch develop
git push -u origin develop
```

## Feature Branches

Each new feature should reside in its own branch, which can be pushed to the central repository for backup/collaboration. But, instead of branching off of master, feature branches use develop as their parent branch. When a feature is complete, it gets merged back into develop. Features should never interact directly with master.

### Create a feature branch

```bash
git checkout -b feature/MYFEATURE develop
```

### Share a feature branch

```bash
git checkout feature/MYFEATURE
git push -u origin feature/MYFEATURE
```

Continue your work and use Git like you normally would.

### Get latest for a feature branch

```bash
git checkout feature/MYFEATURE
git pull --rebase origin feature/MYFEATURE
```

### Finalize a feature branch

When you’re done with the development work on the feature, the next step is to merge the feature/MYFEATURE into develop.

```bash
git checkout develop
git merge --no-ff feature/MYFEATURE
git push

git branch -d feature/MYFEATURE
git push origin :feature/MYFEATURE (if pushed)
```

## Release Branches

Making release branches is another straightforward branching operation. Like feature branches, release branches are based on the develop branch. Once it's ready to ship, the release branch gets merged into master and tagged with a version number. In addition, it should be merged back into develop, which may have progressed since the release was initiated.

### Create a release branch

```bash
git checkout -b release/0.1.0 develop
```

### Share a release branch

```bash
git checkout release/0.1.0
git push -u origin release/0.1.0
```

Continue your work and use Git like you normally would.

### Get latest for a release branch

```bash
git checkout release/0.1.0
git pull --rebase origin release/0.1.0
```

### Finalize a release branch

```bash
git checkout master
git merge --no-ff release/0.1.0
git tag -a 0.1.0
git push
git push origin --tags

git checkout develop
git merge --no-ff release/0.1.0
git push

git branch -d release/0.1.0
git push origin :release/0.1.0 (if pushed)
```

## Hotfix Branches

Maintenance or “hotfix” branches are used to quickly patch production releases. Hotfix branches are a lot like release branches and feature branches except they're based on master instead of develop.

As soon as the fix is complete, it should be merged into both master and develop (or the current release branch), and master should be tagged with an updated version number.

### Create a hotfix branch

```bash
git checkout -b hotfix/0.1.1 master
```

### Share a hotfix branch

```bash
git checkout hotfix/0.1.1
git push -u origin hotfix/0.1.1
```

Continue your work and use Git like you normally would.

### Get latest for a hotfix branch

```bash
git checkout hotfix/0.1.1
git pull --rebase origin hotfix/0.1.1
```

### Finalize a hotfix branch

```bash
git checkout master
git merge --no-ff hotfix/0.1.1
git tag -a 0.1.1
git push
git push origin --tags

git checkout develop
git merge --no-ff hotfix/0.1.1
git push

git branch -d hotfix/0.1.1
git push origin :hotfix/0.1.1 (if pushed)
```