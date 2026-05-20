# Cutting a THAPI release

Example: `0.0.13 -> 0.0.14`. Work from a clean checkout.

## 1. Bump on devel

In `configure.ac`:

```diff
-AC_INIT([thapi],[0.0.13],[bvideau@anl.gov])
+AC_INIT([thapi],[0.0.14],[bvideau@anl.gov])
```

```sh
git checkout devel && git pull --ff-only
# edit configure.ac
git commit -am "0.14"
git push origin devel
```

`configure.ac` is the only version pin in this repo.

## 2. Fast-forward master + tag

```sh
git checkout master && git pull --ff-only
git merge --ff-only devel
git tag -a v0.0.14 -m "0.0.14"
git push origin master v0.0.14
```

Always `--ff-only`. If it refuses, master diverged from devel —
reconcile before continuing.

## 3. Update THAPI-spack

In `~/THAPI-spack/`, add one line to `packages/thapi/package.py`:

```diff
     version("develop", branch="devel")
+    version("0.0.14", tag="v0.0.14")
     version("0.0.13", tag="v0.0.13")
```

Then audit `depends_on(...)`. The new version is auto-covered by
ranges like `@0.0.13:master`, so most releases need no further edit.
The one thing worth scanning for: a constraint scoped to `@master` or
`@develop` that the new release now satisfies. Fold it into the
release-spanning range and delete the carve-out.

PR it:

```sh
git checkout -b release/thapi-0.0.14
git commit -am "Add thapi@0.0.14"
git push -u origin release/thapi-0.0.14
gh pr create --title "Add thapi@0.0.14" --body "Tracks v0.0.14 release."
```
