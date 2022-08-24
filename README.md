# Boon apps release flow proposal

This repo is intended to demo a potential release flow, where most actions is automated based on git state.

## Goals

These are the prioritized aims of the flow.

1. **Convenient** - Any addition must be a net win in productivity, at the very least removing as much hassle as it adds.
2. **Automatic** - If an action _can be_ automatic, it _should be_.
3. **Legacy Support** - It should be easy and fast to support legacy major & minor versions of _all_ apps.
4. **Transparent** - Ideally, the flow would also take care of keeping changelogs and posting on Slack, to let the team know what's moving.

## Git visualization

Let's try and visualize a few practical use cases, building on top of one-another!

### Case-study: A simple release without issues

Let's begin by visualizing a simple release.

```mermaid
gitGraph
  checkout main
  commit id: "feat: a"
  commit id: "feat: b"

  checkout main
  branch release/1.1.x order: 0
  commit id: "chore: bump (1.1.0)" tag: "group-coaching-app@1.1.0"

  checkout main
  commit id: "feat: c"
```

In the above example, we see features being pushed to the `main` branch, and when the feature-set is ripe for release, the `release/1.1.x` branch is made for the release to live in.
A single version bump commit it pushed to the `release/1.1.x` branch, and then a tag is created at that point to represent an actual release.

### Case-study: A bug is discovered that needs to be patched in 1.1.x

Now, let's say that a bug is discovered, and we need to create a patch for it.
Building upon the previous visualization we might get something like this:

```mermaid
gitGraph
  checkout main
  commit id: "feat: a"
  commit id: "feat: b"

  checkout main
  branch release/1.1.x order: 0
  commit id: "chore: bump (1.1.0)" tag: "group-coaching-app@1.1.0"

  checkout main
  commit id: "feat: c"

  checkout release/1.1.x
  branch fix/some-bug-a order: 11
  commit

  checkout release/1.1.x
  merge fix/some-bug-a
  commit id: "chore: bump (1.1.1)" tag: "group-coaching-app@1.1.1"

  checkout main
  merge fix/some-bug-a

  checkout main
  commit id: "feat: d"
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
  commit id: "feat: a"
  commit id: "feat: b"

  checkout main
  branch release/1.1.x order: 0
  commit id: "chore: bump (1.1.0)" tag: "group-coaching-app@1.1.0"

  checkout main
  commit id: "feat: c"

  checkout release/1.1.x
  branch fix/some-bug-a order: 11
  commit

  checkout release/1.1.x
  merge fix/some-bug-a
  commit id: "chore: bump (1.1.1)" tag: "group-coaching-app@1.1.1"

  checkout main
  merge fix/some-bug-a

  checkout main
  commit id: "feat: d"
  commit id: "feat: e"

  checkout main
  branch release/1.2.x order: 1
  commit id: "chore: bump (1.2.0)" tag: "group-coaching-app@1.2.0"
```

There is no more novel information to add here.

### Case-study: A new bug is discovered, and it affects both `1.1.x` and `1.2.x`

This case is again similar to an earlier case, here it is the earlier bug fix case.

```mermaid
gitGraph
  checkout main
  commit id: "feat: a"
  commit id: "feat: b"

  checkout main
  branch release/1.1.x order: 0
  commit id: "chore: bump (1.1.0)" tag: "group-coaching-app@1.1.0"

  checkout main
  commit id: "feat: c"

  checkout release/1.1.x
  branch fix/some-bug-a order: 11
  commit

  checkout release/1.1.x
  merge fix/some-bug-a
  commit id: "chore: bump (1.1.1)" tag: "group-coaching-app@1.1.1"

  checkout main
  merge fix/some-bug-a

  checkout main
  commit id: "feat: d"
  commit id: "feat: e"

  checkout main
  branch release/1.2.x order: 1
  commit id: "chore: bump (1.2.0)" tag: "group-coaching-app@1.2.0"

  checkout release/1.1.x
  branch fix/some-bug-b order: 11
  commit
  commit
  commit

  checkout release/1.1.x
  merge fix/some-bug-b
  commit id: "chore: bump (1.1.2)" tag: "group-coaching-app@1.1.2"

  checkout release/1.2.x
  merge fix/some-bug-b
  commit id: "chore: bump (1.2.1)" tag: "group-coaching-app@1.2.1"

  checkout main
  merge fix/some-bug-b

  checkout main
  commit id: "feat: f"
  commit id: "feat: g"
```

So again, the `fix/` branch is based on the _earliest supported version with the issue_, and when done, is merged into _that same and all following versions affected by that same issue_.

This way, we should be able to easily patch old versions that are still adequately used by the users to warrant support.

### Case-study: Adding cherry-picking

Let's examine how this would look of we were to fix issues on the `main` branch and then use cherry-picking for distribution to `release/` branches.

```mermaid
gitGraph
  checkout main
  commit id: "feat: a"
  commit id: "feat: b"

  checkout main
  branch release/1.1.x
  commit id: "chore: bump (1.1.0)" tag: "group-coaching-app@1.1.0"

  checkout main
  commit id: "feat: c"

  checkout main
  commit id: "fix: a"

  checkout release/1.1.x
  cherry-pick id: "fix: a"
  commit id: "chore: bump (1.1.1)" tag: "group-coaching-app@1.1.1"

  checkout main
  commit id: "feat: d"
  commit id: "feat: e"

  checkout main
  branch release/1.2.x
  commit id: "chore: bump (1.2.0)" tag: "group-coaching-app@1.2.0"

  checkout main
  commit id: "fix: b"

  checkout release/1.1.x
  cherry-pick id: "fix: b"
  commit id: "chore: bump (1.1.2)" tag: "group-coaching-app@1.1.2"

  checkout release/1.2.x
  cherry-pick id: "fix: b"
  commit id: "chore: bump (1.2.1)" tag: "group-coaching-app@1.2.1"

  checkout main
  commit id: "feat: f"
  commit id: "feat: g"
```

The most immediate difference is that the `fix/` branches are missing. Consider that the merging would usually occur through squash, as such in the earlier cases the `fix/` branches should now be considered a permanently visible part of the history.
