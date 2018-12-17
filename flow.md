---
layout: default
---

# Yetamine Flow

Yetamine Flow reflects semantic versioning and defines a branching model that maintains an appropriate multi-version tree structure, which helps preventing situations where semantic versioning rules could be violated. As such, it is suitable for developing components that have a public API which must be versioned properly. It is not suitable so much for aggregate components whose versioning may be arbitrary in the principle (often the case of whole applications and products).


## Versioning terminology

[Semantic versioning](http://semver.org/) defines a **version number**, or colloquially just a **version**, as a composite consisting of three numeric components: *major*`.`*minor*`.`*patch* (which has other names, e.g., *micro*). Version numbers can be compared and form a total ordering. Such a total ordering allows making well-defined **version intervals**, or **version ranges**.

Version intervals are used in semantic versioning to indicate version compatibility. [Semantic Versioning Technical Whitepaper](http://www.osgi.org/wiki/uploads/Links/SemanticVersioning.pdf) from [OSGi Alliance](http://www.osgi.org/) describes this topic in deep details, and although the details are described for the case of the OSGi framework, the underlying concepts and ideas are applicable in other contexts.

Some version intervals are more interesting from the point of view of our branching model:

* A **major version interval** has the form of *[n.0.0, m.0.0)* where *m* = *n* + 1.
* A **minor version interval** has the form of *[a.n.0, a.m.0)* where *m* = *n* + 1.

Both interval types are left-closed and cover all version numbers with the same prefix, major or minor, as the left interval bound.


## Branching model

A branch associated directly with a version information is called a **version branch**. Version branches use the `version/` prefix followed with the associated major or minor version interval or with the exact version. The specific version branches are called accordingly **major version branch** or **minor version branch** for the case of version intervals, but **release version branch** for the case of an exact version. Version branches form the core of the branching model.

Let's start with an example that depicts the usual structure of a branch tree:

```
R ─ ROOT
│
S
│
C1 ─ FORK/1.x                       // Starting with version 1.x
│
C2 ──┬─ STABLE/1.x                  // Forking version 1.0.0 for releasing
│    │
C4   C3 ─ RELEASE/1.0.0
│    ↑
C5   version/1.0.0
│
C6 ──┐                              // Forking version 2.x
│    │
C5   C7 ─ FORK/2.x                  // Starting with version 2.x
│    │
C5   C8 ──┬─ STABLE/2.x             // Forking version 2.0.0 for releasing
│    │    │
C18  C10  C9 ─ RELEASE/2.0.0
│    │    ↑
C19  C11  version/2.0.0
│    │
C20  C12 ─┐                         // Forking version 2.1.0 for releasing
│    │    │
C21  C14  C13 ─ RELEASE/2.1.0
│    │    ↑
C22  C15  version/2.1.0
│    │
C23  C16 ─────────┐                 // Forking version 3.x
│    ↑            │
C24  version/2.x  C17 ─ FORK/3.x    // Starting with version 3.x
↑                 ↑
version/1.x       version/3.x
```

Note that the major version branches use *n*`.x` syntax to indicate major version interval starting with version *n*.0.0.


### Major version branches

Major version branches are the main branches of the whole tree. Instead of the common `master` branch tracking a linear history, every major version branch represents the part of the history associated with its version interval. When a reason for a major version change occurs, a new major version branch should be created in order to track the history of the next major version. This enables parallel development of multiple versions.


### Release version branches

A release version branch is associated with a particular release, or **release version**, and serves to capture the state for the release build.

When a release should be made, a release version branch associated with an appropriate version forks from a source branch. Note that releases of different major versions are independent, so releases are always handled within the scope of a particular major version.

Let's now assume the typical and preferred case when all the releases are forked always from the head of the particular major version branch. The releases are then ordered along with the history of the major version branch. The first release of a major version *n*`.x` shall be assigned version *n*.0.0. Each release sets the baseline version that affects the next release. The code changes since the previous release determine whether the baseline version needs a patch (micro) or minor number bump.

Note that the baseline until the first release does not exist and therefore any changes are allowed (even breaking changes). The first release turns the major version branch into the *stable* state after which breaking changes lead to forking a new major version branch.


### Hotfix branches

The release version branch description included the regular case when the order of creating releases matches the release version ordering. When a bug appears, the recommended approach is fixing the bug during as a regular task and include the fix in the next release that shall be compatible with the buggy one. Projects keeping semantic versioning strictly should have no problem with such an approach.

However, the recommended approach might not be acceptable in some cases and then a hotfix branch may be useful. The hotfix branch forks from the same commit as the release version branch to fix and it inherits the release version. If multiple hotfixes are applied for the same release version, they share the original hotfix branch. Particular hotfix release have their own release version branches as usual.

Let's assume that in our example version 2.0.0 suffers from a bug and it needs a hotfix. However, the hotfix does not work properly and another hotfix must be applied. The figure below shows the version 2.x part of the branch tree after both hotfixes are released. Hotfix branches use `hotfix/` name prefix:

```
                                        ⁞
                                        C7 ─ FORK/2.x
                                        │
                         ┌───────────── C8 ──┬─ STABLE/2.x
                         │              │    │
                  ┌───── C25            C10  C9 ─ RELEASE/2.0.0
                  │      │              │    ↑
RELEASE/2.0.1 ─ C26      │              C11  version/2.0.0
                  ↑      │              │
      version/2.0.1      │              C12 ─┐
                         │              │    │
                  ┌───── C27            C14  C13 ─ RELEASE/2.1.0
                  │      ↑              │    ↑
RELEASE/2.0.2 ─ C28      hotfix/2.0.0   C15  version/2.1.0
                  ↑                     ⁞
      version/2.0.2
```

Note that hotfixes are an emergency measure and should be avoided when possible. Moreover they are limited to patch version numbers and therefore may not change too much (anyway, changes beyond the scope of the patch version number are hardly hotfixes, right?). Even then, multiple hotfixes may require more care: imagine a situation when a hotfix for version 3.0.1 should be issued, while version 3.0.2 already exists. Then the hotfix must get the next free patch version number, e.g., 3.0.3, which looks confusing and may bring other problems later. Proper naming of all the branches discovers the version clash early at least.


### Remarks

Minor version branches were mentioned in the branching model introduction, but the example shows none and no details were given so far. In principle, minor version branches are possible. They would be named `version/`*n*`.`*m*`.x` and provide similar isolation of minor version intervals like major version branches do in the case of major version intervals. However, such an isolation is too fine-grained and hardly useful in practice. (Technically, hotfix branches are a kind of minor version branches, but they are created due to extraordinary needs out of normal flow.)

The requirement of version interval consistency imposes strong constraints on the branching model and basically tells what may or may not happen. However, the consistency must be judged with the respect to actual releases. For instance, two completely independent branches of the same functionality may exist, offering two incompatible altenatives; when one of them is released, it reserves a version interval and the other branch must choose a different one. Hence the release introduced new constraints for both existing and future branches.


### Tags

The example shows several tags related to the branching model with various roles. Let's explain them.

* `FORK/`*n*`.x` marks the first actual commit on `version/`*n*`.x` branch. `FORK` tags make easier finding the point where the development of any (major) version starts. The tags may be annotated and therefore carry the details about the reasons for forking the version branches etc. (Git notes is another alternative to attach the details of the fork.)

* `STABLE/`*n*`.x` marks the commit on `version/`*n*`.x` since which no breaking changes are allowed, i.e., when the branch turns into the *stable* state. Usually the tag marks the commit where `release/`*n*`.0.0` branches off, but it may occur before that commit (e.g., when a release candidate milestone passed).

* `RELEASE/`*a*`.`*b*`.`*c* marks the release with release version *a.b.c*. Because these tags represent a published state, it is generally a good idea to sign them. The tag message provides the opportunity to attach release notes, change log since the previous release, or other important information.


### Development branches

The branch types described above are not meant for actually committing the code during development, but rather to maintain the history and organize the version tree structure. A **development branch** is the proper branch type to receive a developer's commits. Some people want to distinguish more types of the development branches, e.g., to have specific feature and bugfix branches, but all these variants behave similarly in principle.

The differences of handling the development branches result in variants of the workflow. Here are described the two major variants.


#### *Deliver features*

The developer forks a dedicated branch for the next functionality to deliver. The branch is usually linked to a ticket, e.g., a bug report or feature request. When the functionality is finished, the branch is merged back.

The development can proceeds on the forked branch independently and more developers can work on it. The painful part often is merging the changes back, after which the forked branch should be finished. When other merges between the forked and source branches happen, the nice isolation of the functionality development is lost and complicated situations may occur.

The painful part with merging the changes back can be mitigated by rebasing the forked branch before merging it back, although this brings other pains. Another question is how to deal with the finished branches: should they be preserved or deleted?

Familiar commonly used name prefix for the forked development branch is `feature/`.


#### *Deliver to version*

The workflow resembles *deliver features*, but the development branch remains local and therefore rebasing it (even frequently) brings no pain. Fast-forward merging then hides completely that the commits didn't land on the major version branch directly, hence the name of the variant.

When each functionality can be developed by a single developer, this workflow variant could be preferred as it brings no pain and overhead.


## Recommendations

There are a few general recommendations which are not related directly to a particular workflow, but using them (especially with a more complex workflow like Yetamine Flow) helps:

* Establish a good commit message policy. See [Pro Git](https://git-scm.com/book/) for commit guidelines.
* Use interactive rebasing to make your commits ordered and clean before publishing them.
* Prefer fast-forward merging to keep the history concise.

Following points should keep the repository structure better understandable at a glance:

* Names of tags and branches have hierarchical structure with `/` (slash) as the component separator.
* Branch names should follow *lower-case* convention.
* Tag names should follow *UPPER-CASE* convention.

Using *UPPER_CASE* convention with `_` (underscore) might be a more natural choice, it is mostly the matter of the convention. Having the same word separator for tags and branches makes them more consistent. However, tag names are commonly used in artifact version qualifiers, which usually use a dash to separate their parts.


### Special commits

The example shows two special commits marked as `R` and `S`. They are not essential for the branching model, but they provide some benefits and the idea can be used basically with any workflow. So why not to use them?

The **root commit** (marked as `R` in the example) must be empty and must be the least common ancestor of all branches in the repository. A root commit provides a non-conflicting common root for merging or rebasing any branch, which makes splitting and merging whole repositories significantly easier. The root commit should be marked with `ROOT` tag and it may contain the information about setting up the repository, used workflow etc.

The **setup commit** should provide the stub for starting the work with the repository. The setup depends on team policies and rules, but the common content includes `.gitignore`, `.gitattributes` and `README.md` or similar document describing the purpose of the repository. Having multiple commits instead of a single one is a possibility to consider as well. The usual `master` branch may be preserved pointing to the last setup commit. Then the major version branch(es) can fork from the `master` branch naturally.


### When to fork

The branch structure provides quite a lot of hints when a new version branch should be forked and what version range constraints should be applied. Before committing a change, the change must be considered with the respect to the version range constraints of the target branch. If the change violated the constraints, it may not be committed to the branch and a new version branch of an appropriate version range must be forked instead to accept the change.

To determine if a change needs forking the target branch it is enough to determine the baseline version (i.e., the version number of the latest release in the history of the target branch) and collect the scope of all changes, including the change to be committed (i.e., if the major, minor or micro version number needs updating). These two parameters produce the version number for the change to be committed, if it should be released. If the version number violates the constraints of the target branch, the branch needs a fork for the change.


### Forking traps

Forking a version brings usually some problems, at least if the artifact aggregates multiple versioned items, and it does not matter if creating a new master branch or just a fix branch. A fix branch, which should change implementation details only (i.e., some private code) and therefore influence just the micro version number, can still bring problems.

For instance, let's have an OSGi bundle in version 1.5.2 with packages *foo* in version 1.2.0 and *bar* in version 1.3.0 which is about to change the major version; this forces the artifact to fork a new version 2.0.0 that contains the *foo* package, without any change, and *bar* with the new major version 2.0.0. However, any further development of the *foo* package becomes troublesome, because changes of the package on both branches must be synchronized in order to maintain consistent versioning.

There are some acceptable solutions (although not without pain):

* Separating the common packages in another artifact. When taking this choice, the common packages probably provide a well-defined functionality that can exist on its own. Making a new project for further development of them usually is a good idea anyway, which solves the synchronization problem.

* Having the same version for all packages. Forking a branch results in forking the packages as well, keeping them independent and synchronized with the branch. This choice makes sense when the packages form a consistent group, which shall be used together, and when more of those packages are affected by major changes; then even if a package from the group is not changed, changing its version number might be an acceptable price for re-releasing the actually same code under a new version number.

* Having a package master branch. The stable packages are maintained on the current branch and their changes are merged in the other branches. This choice is difficult to manage, but possibly still acceptable when there are few such packages and they are stable. Using tools for managing the changes and versions is recommended then, e.g., the slave branches might have restrictions on the package changes that force using the merges, instead of any development of the package on the slave branch.

* Limiting existing branches on fork. Forking a new version limits existing branches strictly to prevent conflicts; existing branches still may continue in the maintenance mode, but can't advance beyond the limits. As a result, the version tree may become (almost) linear. To restore the parallelism, the incubation phase of the new version should be long enough to accumulate more changes and hopefully more packages switch their major version meanwhile too.


## How to…

This section gathers the recommendations, procedures and examples for the most usual tasks.


### Create a new repository

1. Make a new empty repository.
2. Make the root commit: `git commit --allow-empty -m "Make the root commit"`
3. Make the root commit tag: `git tag ROOT`
4. Make the setup commit(s).

Usually this step is followed by making `version/1.x` branch, while `master` branch may be deleted, or preserved to mark the last setup commit.


### Create a major version branch

If a major version branch forks from the root commit or from a setup commit, it begins an independent version tree. It is a good practice to commit, as the first commit of the branch, the skeleton of the intended content, e.g., an empty, but already compilable project with a name and core dependencies for testing etc.

A major version branch forks from another version branch typically when a change could not be committed on the original branch due to its version range constraints. The change then comes to the new branch, usually as the first commit of the new branch.

The fork commit (i.e., the commit containing the skeleton or the incompatible change) should be tagged: `git tag FORK/`*n*`.x`


### Make a release

Let's assume that the source branch contains the desired state for a release. The source branch usually is a version branch, although it may be a hotfix branch for a hotfix release.

Because the release consists of multiple steps and possibly multiple commits (depending on the way of tracking and applying the version numbers), the procedure must not be interrupted and withing a version range of a version branch only one release can be made at a time.

1. Determine the target version. It shall become the new version baseline for next changes on the source branch.
2. Update the version baseline and constraints for next changes on the source branch if necessary.
3. Check that the build still can pass all the criteria for the release.
4. Commit the changes: `git commit -m "Update to version `*a.b.c*`"`
5. If the source branch is yet unstable, mark it as stable from now on: `git tag STABLE/`*n*`.x`
6. Fork the release version branch: `git checkout -b version/`*a.b.c*
7. Set the version number for the final artifact to the target version.
8. Perform a clean full build, check the output and stage it for publishing.
9. If the release is approved, commit the release: `git commit -m "Release version `*a.b.c*`"`
10. Make the release version tag: `git tag -sm "Release version `*a.b.c*`" RELEASE/`*a.b.c*
11. Publish the code (i.e., push the changes to the upstream) and publish the artifacts.

If the procedure fails at some point, revert the changes, fix the problem and retry. Until all changes are published in the last step, it can be done. The procedure therefore reminds a two-phase commit: it prepares everything up to the point when the commit (i.e., publishing the release) is safe.


### Publish a snapshot

Dependencies on snapshot builds are used conveniently during development, e.g., in multi-module projects where the modules have a shared lifecycle. However, some projects may be more loosely coupled, yet having a kind of mutual relationship. For instance, project *A* is an independent library that is being developed primarily to support project *B*, therefore *A* development tends to react rapidly on demands of project *B* and project *B* usually wants to try latest features and updates of project *A*.

However, letting project *B* depend on the latest build of project *A* might not be the best idea: the builds may change too frequently and completely out of any control on the side of project *B*, which may break its builds randomly and unpredictably. Using automatic timestamps of snapshot builds (which, e.g., Maven supports) has the disadvantage that it is not reliable.

But what about performing a simplified release procedure for a snapshot? Such a published snapshot does not promise the stability of an actual release, but it is perfectly reproducible unlike the usual snapshot build (which actually means the latest build). The procedure could consist of following steps:

1. Fork the snapshot branch from the source branch with the given *version* (including a unique part): `git checkout -b snapshot/`*version*
2. Set the version number for the artifact to *version*.
3. Commit the changes: `git commit -m "Publish snapshot `*version*`"`
4. Make the snapshot version tag: `git tag -sm "Publish snapshot `*version*`" SNAPSHOT/`*version*
5. Publish the code (i.e., push the changes to the upstream) and publish the artifacts.

When any snapshot can be built in the way that the SHA1 hash of the source commit appears in *version*, nothing else must be actually committed and the build can still be reproduced for each commit. On the other hand, the authors might use `SNAPSHOT` tags to indicate versions suitable for external evaluation (perhaps with more details in an annotation) and the tags might be used instead of the hashes. The advantage of the tag-based approach is that it is more explicit and it does not break when the history is rewritten carefully (e.g., when a commit includes some data that does not affect the build, but that is sensitive and should be deleted from the history anyway).
