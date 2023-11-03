---
layout: default
redirect_from: /docs/multi-version-flow/
---

# Multi-Version Flow

Multi-Version Flow is a workflow that supports multiple versions in parallel. Following text describes this workflow for Git, but this workflow can be adapted for almost any version control system that supports branching.


## Motivation

Multi-Version Flow embraces version management complexity and defines general rules to cope with all valid situations, instead of defining special rules, which other workflow often do.

Proper use of Multi-Version Flow requires familiarity with version management, but does not have to be difficult. It consistently follows rules that respect versioning naturally and that makes using it easier when the user understands the concepts behind.


## Versioning terminology

[Semantic versioning](http://semver.org/) defines a **version number**, or colloquially just a **version**, as a composite consisting of three numeric components: *major*`.`*minor*`.`*patch* (which has other names, e.g., *micro*). Version numbers can be compared and form a total ordering. Such a total ordering allows making well-defined **version intervals**, or **version ranges**.

Following text uses the common interval notation where square brackets indicate inclusive bounds and round brackets indicate exclusive bounds.

Some version intervals are more significant than others and deserve special attention:

- A **major version interval** stands for `[n.0.0, m.0.0)` where `m = n + 1`.
- A **minor version interval** stands for `[x.n.0, x.m.0)` where `m = n + 1`.

For short, a major version interval `[n.0.0, m.0.0)` can be called also major version `n` and expressed as version `n.x`. For instance, version `1.x` stands for major version interval `[1.0.0, 2.0.0)`. Minor versions are defined similarly. For instance, version `1.2.x` stands for `[1.2.0, 1.3.0)`.

Exact versions are trivially equivalent to version intervals that contain only one version, hence interval `[x.y.z, x.y.z]` stands for version `x.y.z`.

A bare version number, defined above, may be extended with metadata. Semantic versioning defines pre-release labels as the metadata that affect version number comparison. An extended version number with a pre-release label has lower precedence than the normal version number. We call all such versions collectively as **snapshots**.


## Branching model

A branch associated directly with a version interval is called a **version branch**. Version branches use `version/` prefix followed with the associated version using the short notation. For instance, `version/1.x` stands for the major version branch `1.x` that covers version interval `[1.0.0, 2.0.0)`.

Version branches form the core of the branching model. They track  version evolution while keeping version numbers consistent. Every version may appear only once and it must be included in the version interval of its branch.

Version branches fork from other version branches as the result of natural evolution. If a version branch forks from another version branch, its version interval must be either confined within its parent branch version interval, or it must not overlap with any existing one. The former case is typical for minor or exact version branches, while the latter case is typical for major version branches.

The first version branch can't naturally fork from another version branch. Instead, the `main` branch exists as the common root for the version branches that do not fork from other version branches. The typical case, except for the first branch, is starting a new major version from scratch instead of forking from an existing version branch.

Here is an example that depicts a usual structure:
```
R = ROOT
│
S ───┬── C21
↑    │   ↑
main C1  version/3.x
     │
     C2 ─ C3 = RELEASE/0.1.0
     │    ↑
     C4   version/0.1.0
     │
     C5 ─────┐
     ↑       C6 ── C7 = RELEASE/1.0.0
version/0.x  │     ↑
             C8    version/1.0.0
             │
             C9
             │
             C10 ─ C11 ─ C12 = RELEASE/2.0.0
             │     │     ↑
             C15   C13   version/2.0.0
             │     │
             C16   C14
             │     ↑
             C17   version/2.x
             │
             C18 ─ C20 = RELEASE/1.1.0
             │     ↑
             C19   version/1.1.0
             ↑
             version/1.x
```

This graph shows following notable events:

- There are four major versions: `0.x`, `1.x`, `2.x`, and `3.x`.
- There are four releases so far: `0.1.0`, `1.0.0`, `1.1.0` and `2.0.0`.
- Version `0.x` looks like not developed anymore, while other major versions have some trailing commits, which indicates that these branches are still maintained.
- Version `1.x` forked from `0.x`.
- Version `2.x` forked from `1.x`.
- Version `3.x` forked from `main`, thus starting from scratch rather than merely evolving from a previous version.

Although example retains all version branches for didactic purposes, it may be a good practice to remove version branches that are not used anymore for maintenance or development. An absence of such a branch is a good indicator that the development finished.

In the case of exact version branches, this practice makes sense even more and helps to reduce the branch list significantly. Exact version branches should exist just during the release process and be removed after tagging the release.

At the end, release tags provide enough information to recover the version branch structure if necessary, therefore removing obsolete version branches is recommended.


### The `main` branch

The `main` branch should be the common parent for all version branches. Conveniently, the default branch created by the versioning system can be then a natural starting point for forking actual version branches.

The example shows two special commits marked as `R` and `S`. Although these commits are not essential for the branching model, they provide some benefits. So why not to use them?

The **root commit** (marked as `R` in the example) must be empty and should be the least common ancestor of all branches in the repository. A root commit provides a non-conflicting common root for merging or rebasing any branch, which makes splitting and merging even whole repositories significantly easier. The root commit should be marked with `ROOT` tag and it may contain the information about setting up the repository, used workflow etc. in its annotation.

The **setup commit** should provide the stub for forking version branches. The setup depends on team policies and rules. It is usually better to have the setup minimalistic. A usual setup commit includes `.gitignore`, `.gitattributes` and `README.md` or similar document describing the purpose of the repository. Having multiple commits instead of a single one is a possibility to consider as well.


### Version branches

Major version branches form the core of a repository and provide  natural focus points for ongoing activities. They are the only long-term branches defined by the Multi-Version Flow, although even they have lifetime limited by the activities on their versions.

Exact version branches serve for creating releases. They should be removed after tagging the release to keep the branch list clean. A release branch can be still convenient for the release process. In most cases, an exact version branch forks from a major version branch.

Minor version branches are possible, but having limited usefulness. A minor version branch may be useful for reserving a version interval, so that all releases within this interval must fork from the branch. Such a measure may be useful for maintaining multiple hotfixes when the next minor version was already released, which blocks branching subsequent hotfixes for the previous minor version safely from the major branch.

There is no need for hotfix branches, common in other workflows, because an exact version branch for a hotfix can be forked from convenient commits anywhere if the version fits into the parent version branch's version interval.


### Development branches

Version branches are meant primarily as management measures that help to coordinate and integrate individual contributions towards releases, while keeping the repository structure clean, lean and organized.

Regular development activities should use additional short-living branches that fork from version branches and merge back the work done on them. Having such branches help isolating concurrent activities as well as organizing the commits to be merged back.

Multi-Version Flow does not define the organization of these branches. However, these branches are commonly called feature branches and the 
usual naming convention uses `feature/` prefix for them.


### Releases

A release process, as noted before, can employ exact version branches. Releases themselves do not depend on them though and may be created at any commit as long as the version consistency is preserved.

The standard convention proposes release tag form of `RELEASE/x.y.z` to mark the commit for release `x.y.z`. It is generally a good idea to sign these tags and to attach annotations with additional details, like the change log or release notes.


### Snapshots

Snapshots are not visible a lot in the branching model, but play an important role. There are two kind of snapshots. Their handling and lifecycle differ significantly.

A **moving snapshot** may be defined for an active branch. It is bound to the tip of that branch, therefore it is moving with the branch and it is inherently unstable.

It is convenient to accept implicit moving snapshots for major and minor version branches. The version of the snapshot should be then the upper bound of the branch's version interval with a suitable extending label. For instance, the moving snapshot for the `version/1.x` branch is `2.0.0-SNAPSHOT` (because `2.0.0` is the upper bound of `version/1.x`). Note that the upper bound is the convergence point of the version branch, while the label reduces the precedence of the snapshot, so that the actual release of the upper bound version prevails eventually.

Moving snaphots with upper bounds are problematic when using a build system that does not support version qualifiers. If the scope of the build can be limited, using the upper bound directly as the version may be acceptable, otherwise `0.0.0` might be a convenient substitute, because it can be interpreted as an undefined version and it usually needs some effort to actually use an artifact with such a version in any version-aware environment, reducing changes of an unintended use.

A **stable snapshot** captures the state of a specific commit. This makes it very similar to a release, just without its stability guarantees. Making a stable snapshot can follow the usual release procedure with minor modifications to distinguish stable snapshots from releases by using different names for tags and branches. For instance, `SNAPSHOT/` instead of `RELEASE/` and `snapshot/` instead of `version/`.


## Recommendations

There are a few general recommendations which are not related directly to a particular workflow, but using them helps:

* Establish a good commit message policy. See [Pro Git](https://git-scm.com/book/) for commit guidelines.
* Use interactive rebasing to make your commits ordered and clean before publishing them.
* Prefer fast-forward merging to keep the history concise.

Following points should keep the repository structure better understandable at a glance:

* Names of tags and branches should have a hierarchical structure with `/` (slash) as the separator between hierarchy levels.
* Branch names should be in lower case.
* Tag names should be in upper case.

Using `-` (dash) or `_` (underscore) as the word separator in branch and tag names is mostly the matter of a convention. Using always the same separator improves consistency and removes the need to decide when to use which separator. Dashes are probably used more frequently in this context, therefore look like the more natural choice.

Regarding versions, it is generally a good idea to specify the current working version a moving snapshot and use the most appropriate version schema that the used development environment accepts. Preferably, the version should be the upper bound of the version branch extended with a snapshot qualifier.


## How to…

This section gathers the recommendations, procedures and examples for the most usual tasks.


### Create a new repository

1. Make a new empty repository.
2. Make the root commit: `git commit --allow-empty -m "Make the root commit"`
3. Make the root commit tag: `git tag ROOT`
4. Make the setup commit(s).

Usually this step is followed by making `version/0.x` branch.


### Create a major version branch

If a major version branch forks from the `main` branch, it begins an independent version tree. It is a good practice to commit, as the first commit of the branch, the skeleton of the intended content, e.g., an empty, but already compilable project with a name and core dependencies for testing etc.

A major version branch forks from another version branch typically when a change could not be committed on the original branch due to its version interval constraints. For instance, the change is breaking the compatibility guarantees promised by the given version interval. Then the change must go to the new branch, usually as the first commit of the new branch.

An alternative approach is forking the next major version branch from the first release commit of the major version, which naturally closes the previous version interval and opens the new one. This is natural especially for version `1.0.0` when the inherently unstable `version/0.x` branch loses its further meaning as the project enters in the period when version compatibility must be managed more carefully.


### Make a release

The easiest approach leverages the definition of a moving snapshot for the source branch. The moving snapshot implies that the version can be fixed for the source branch to the upper bound version with a label to mark a snapshot.

1. Make a `version/x.y.z` branch for the intended release version `x.y.z`.
2. If necessary, make last minute changes on the version branch. The typical operation is fixing the version information for the build.
3. If the build passes, the release is tagged with `RELEASE/x.y.z`.
4. The version branch can be deleted.

It is possible to skip making the version branch entirely if no last minute changes are necessary, which reduces the operation to making just the release tag. This makes the release commits natural points from where next versions can continue.

The workflow can be automated too. For instance, a developer makes the version branch and pushes it. A CI/CD pipeline would be then triggered by the push and execute the whole build. Once all release criteria are satisfied, the pipeline can tag the release and delete the version branch. Otherwise it can notify the developer that the release has failed (and eventually keep the branch for investigation). However, for integration with CI/CD, it may be better to use another branch as the trigger point, e.g., using `release/` prefix for such a branch. The version branch can then exist only for explicitly wanted ones.

This procedure can be adapted for making stable snapshots too.
