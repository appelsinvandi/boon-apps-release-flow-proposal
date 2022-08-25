# Boon apps release flow proposal

This repo is intended to demo a potential release flow, where most actions is automated based on git state.

## Goals

These are the prioritized aims of the flow.

1. **Convenient** - Any addition must be a net win in productivity, at the very least removing as much hassle as it adds.
2. **Automatic** - If an action _can be_ automatic, it _should be_ automatic.
3. **Legacy Support** - It should be easy and fast to support legacy major & minor versions of _all_ apps.
4. **Transparent** - Ideally, the flow would also take care of keeping changelogs and posting on Slack, to let the team know what's moving.

## Git visualization

Let's try and visualize a few practical use cases, building on top of one-another!

### Case-study: A simple release without issues

Let's begin by visualizing a simple release.

```mermaid
gitGraph
  checkout main
  commit id:"feat: a"
  commit id:"feat: b"

  checkout main
  branch release/1.1.x order: 0
  commit id:"chore: bump (1.1.0)" tag:"1.1.0"

  checkout main
  commit id:"feat: c"
```

In the above example, we see features being pushed to the `main` branch, and when the feature-set is ripe for release, the `release/1.1.x` branch is made for the release to live in.
A single version bump commit it pushed to the `release/1.1.x` branch, and then a tag is created at that point to represent an actual release.

### Case-study: A bug is discovered that needs to be patched in 1.1.x

Now, let's say that a bug is discovered, and we need to create a patch for it.
Building upon the previous visualization we might get something like this:

```mermaid
gitGraph
  checkout main
  commit id:"feat: a"
  commit id:"feat: b"

  checkout main
  branch release/1.1.x order: 0
  commit id:"chore: bump (1.1.0)" tag:"1.1.0"

  checkout main
  commit id:"feat: c"

  checkout release/1.1.x
  branch fix/some-bug-a order: 11
  commit

  checkout release/1.1.x
  merge fix/some-bug-a
  commit id:"chore: bump (1.1.1)" tag:"1.1.1"

  checkout main
  merge fix/some-bug-a

  checkout main
  commit id:"feat: d"
```

A `fix/` branch is created for the bug on the _lowest supported version where the bug exists_.
In this case that is `release/1.1.x`, so we create `fix/some-bug-a` from that release branch.

Once the bug is fixed, the `fix/` branch is merged into _all relevant branches of the base version **or newer**_.
In this case, that is `release/1.1.x` and `main`, it might happen that the bug simply no longer exists on `main`, in which case it's okay not to merge into `main`.

When the merge into `release/1.1.x` is pushed, package versions are bumped and a new release tag are again created at that commit.

### Case-study: There's features enough for a new minor release

This is handled in a very similar manner to how `1.1.x` were released.

```mermaid
gitGraph
  checkout main
  commit id:"feat: a"
  commit id:"feat: b"

  checkout main
  branch release/1.1.x order: 0
  commit id:"chore: bump (1.1.0)" tag:"1.1.0"

  checkout main
  commit id:"feat: c"

  checkout release/1.1.x
  branch fix/some-bug-a order: 11
  commit

  checkout release/1.1.x
  merge fix/some-bug-a
  commit id:"chore: bump (1.1.1)" tag:"1.1.1"

  checkout main
  merge fix/some-bug-a

  checkout main
  commit id:"feat: d"
  commit id:"feat: e"

  checkout main
  branch release/1.2.x order: 1
  commit id:"chore: bump (1.2.0)" tag:"1.2.0"
```

There is no more novel information to add here.

### Case-study: A new bug is discovered, and it affects both `1.1.x` and `1.2.x`

This case is again similar to an earlier case, here it is the earlier bug fix case.

```mermaid
gitGraph
  checkout main
  commit id:"feat: a"
  commit id:"feat: b"

  checkout main
  branch release/1.1.x order: 0
  commit id:"chore: bump (1.1.0)" tag:"1.1.0"

  checkout main
  commit id:"feat: c"

  checkout release/1.1.x
  branch fix/some-bug-a order: 11
  commit

  checkout release/1.1.x
  merge fix/some-bug-a
  commit id:"chore: bump (1.1.1)" tag:"1.1.1"

  checkout main
  merge fix/some-bug-a

  checkout main
  commit id:"feat: d"
  commit id:"feat: e"

  checkout main
  branch release/1.2.x order: 1
  commit id:"chore: bump (1.2.0)" tag:"1.2.0"

  checkout release/1.1.x
  branch fix/some-bug-b order: 11
  commit
  commit
  commit

  checkout release/1.1.x
  merge fix/some-bug-b
  commit id:"chore: bump (1.1.2)" tag:"1.1.2"

  checkout release/1.2.x
  merge fix/some-bug-b
  commit id:"chore: bump (1.2.1)" tag:"1.2.1"

  checkout main
  merge fix/some-bug-b

  checkout main
  commit id:"feat: f"
  commit id:"feat: g"
```

So again, the `fix/` branch is based on the _earliest supported version with the issue_, and when done, is merged into _that same and all following versions affected by that same issue_.

This way, we should be able to easily patch old versions that are still adequately used by the users to warrant support.

### Case-study: Adding cherry-picking

Let's examine how this would look of we were to fix issues on the `main` branch and then use cherry-picking for distribution to `release/` branches.

![Git Graph](https://mermaid.ink/svg/pako:eNqlk01OwzAQha9izQqkUhovvQZxAHbIm4k9SazGcTS1pVZV745TtaCUhqaw8_g9z8838h5MsAQKahffGPtGd0KYhsw6pCg8uu54Ebx3UTirNFSEUQnUcF0oB-FqkpKxM41gagk39Fwsi-VWBLbESqwuk5kmMOVsyffiYbCuHjWIiHXWjuFkmZ8tmXletz2NNbKO2j0pzLun3pn15cMbIxTjEYo7RrBTuGk-bvmNu7jRqxzjlnNxDyTKvyIsZyCUY4Tyl1ryf7XkeF3yrnVVU-uqNcACPHF-a_Ov2w82DbEhnzep8tFShamNQ4JDtqbeYqRX62JgUBW2G1oAphjed50BFTnR2fTisGb0X64eu48QzvHhExC8Rks)

_Note: GitHub doesn't support cherry-pick in mermaid, so here's an embed. Edit it [here](https://mermaid.live/edit#pako:eNqlk81OwzAMx18l8gmkMdYccwbxANxQLm7ittGapvJSadO0dycdm1DGyjq4xfbfXz8rezDBEiioXXxj7BvdCWEaMuswROHRdUdH8N5F4azSUBFGJVDD9UA5Bq4WKRk70wimlnBDz8WyWG5FYEusxOqymGkCU6o2-F48jNLVowYRsU6xoznZ5udIZp7WbU9rZdJs3FOEeffUO7O-TLyxQpGvUNyxgp3CTfNxy2_cxY1ZZY5bzsU9kij_irCcgVDmCOUvveT_esn8XPKuc1VT56o1wAI8ccq16dftR5mG2JBPl1TpaanCoY1jgUOSDr3FSK_WxcCgIg-0ABxieN915mx_aV4c1oz-7Oyx-wghmRW2Gzp8AooBRbU).

The most immediate difference is that the `fix/` branches are missing. Consider that the merging would usually occur through squash, as such in the earlier cases the `fix/` branches should now be considered a permanently visible part of the history.

Wether or not this is the way to go remains uncertain.

### Case-study: Multiple apps

Alright, so obviously there's more than one app to keep track of. The current branch naming doesn't really allow for this without strong version binding between apps, which is quite unreasonable.

A way to fix this would be to add an extra `folder` to the branch names. Let's take a look at how that would look like, in a scenario without bug-fix noise.

Here is the clean git tree, _without_ any measures to support multiple apps releases.

```mermaid
gitGraph
  checkout main
  commit id:"feat: a"
  commit id:"feat: b"

  checkout main
  branch release/1.1.x order: 0
  commit id:"chore: bump (1.1.0)" tag:"1.1.0"

  checkout main
  commit id:"feat: c"
  commit id:"feat: d"
  commit id:"feat: e"

  checkout main
  branch release/1.2.x order: 1
  commit id:"chore: bump (1.2.0)" tag:"1.2.0"

  checkout main
  commit id:"feat: f"
  commit id:"feat: g"
```

And here it is again _with_ support for multiple apps releases.

```mermaid
gitGraph
  checkout main
  commit id:"feat: a"
  commit id:"feat: b"

  checkout main
  branch release/group-coaching-app/1.1.x order: 0
  commit id:"chore: bump (1.1.0)" tag:"group-coaching-app@1.1.0"

  checkout main
  commit id:"feat: c"
  commit id:"feat: d"

  branch release/group-coaching-coach-dashboard/1.42.x order: 2
  commit id:"chore: bump (1.42.0)" tag:"group-coaching-coach-dahsboard@1.42.0"

  checkout main
  commit id:"feat: e"

  checkout main
  branch release/group-coaching-app/1.2.x order: 1
  commit id:"chore: bump (1.2.0)" tag:"group-coaching-app@1.2.0"

  checkout main
  commit id:"feat: f"
  commit id:"feat: g"
```

Now, the `group-coaching-coach-dashboard` app is a web project, so we don't really have to support it beyond 1 minor version, but this structure will enable us to perform rollbacks when necessary, so it still have some value.

With this structure, the CI will know exactly which app to release for each branch.

## Automation

So by now you're likely thinking, "cool, but that doesn't really improve anything, you just made the branching model slightly more strict and added tags..." You would be right to say this, but it's still a pretty big win for automation. Let's look at some potential flows!

### Case-study: A developer creates a release branch, or a new commit is added to an existing release branch

In this case, a developer actively pushes new code to be released imminently. The CI should then take action to prepare for a release.

```mermaid
graph TD
  Trig_ReleaseBranchCommit([New commit on a release branch])

  Trig_ReleaseBranchCommit
    --> CiCreateReleasePr[CI creates/updates a PR based on that branch,<br/>compiling all changesets into a changelog and bumping package versions]
```

In the above flow, we see that the CI prepares the release in a new PR, which will allow developers review the release before merging the PR.

### Case-study: The developer decides they're happy with the release, and merges the release PR

So the developer looks through the changelog, and makes sure that everything looks as it should. This could also be a good time to make sure the backend is ready for the release. They then decide that it's time to release the new version of the app.

```mermaid
graph TD
  Trig_ReleaseBranchCommit([New commit on a release branch])
  Trig_ReleasePrMerged([A release PR is merged])

  Trig_ReleaseBranchCommit
    --> CiCreateReleasePr[CI creates/updates a PR based on that branch,<br/>compiling all changesets into a changelog and bumping package versions]
    --> DevDecidesToMerge[A developer determines the release is ready,<br/>and merges the release PR]
    -.-> Trig_ReleasePrMerged

  Trig_ReleasePrMerged
    --> CiCreatesReleaseTag[CI creates release tags for all updated apps,<br />with a changelog for each]
```

Through merging the CI can create the git state that signifies a release. Potentially triggering more actions to be explored in the next case-study.

### Case-study: A new release tag is created, and the release should be deployed

With the release tag ready and containing all the relevant release info, a new trigger can act on the created tag.

_Note: As I remember, it's technically not possible to act on a tag created by another GitHub Action. But it can all run in the same GitHub Action. For simplicity's sake, I'll refer to it as a trigger on a new release tag._

```mermaid
graph TD
  Trig_ReleaseBranchCommit([New commit on a release branch])
  Trig_ReleasePrMerged([A release PR is merged])
  Trig_ReleaseBranchTag([New tag based on a release branch commit,<br />is pushed to the remote])

  Trig_ReleaseBranchCommit
    --> CiCreateReleasePr[CI creates/updates a PR based on that branch,<br/>compiling all changesets into a changelog and bumping package versions]
    --> DevDecidesToMerge[A developer determines the release is ready,<br/>and merges the release PR]
    -.-> Trig_ReleasePrMerged

  Trig_ReleasePrMerged
    --> CiCreatesReleaseTag[CI creates release tags for all updated apps,<br />with a changelog for each]
    -.-> Trig_ReleaseBranchTag

  Trig_ReleaseBranchTag
    --> ParseTag[Parse tag information]
    --> InitDeploy[Initialize deployment on relevant platforms]
    --> SlackNotify[A notification is sent to slack about the deployment,<br />with the changelog]
    --> NotionUpdate[CI finds all tickets referenced in the changelog,<br />and moves them to the relevant list on Notion.]
```

So already the last step contains a lot of automation, and there's really a lot of potential to add automation throughout, especially in regards to notion.
