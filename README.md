# Bazel glob expression infinite-recursion bug

This repo demonstrates a bug where an infinite recursion is detected when
using `glob` through a directory traversal that should not happen based on
the `exclude=[]` arguments it is provided.

In this directory, the `foo` subdirectory contains a symlink called `bar`
to the parent directory, which is recursive and thus can cause traversal
problems. `foo` also contains a simple text file.

When we construct a `glob` expression that includes the contents of `foo`
in a wildcard expression, but specifically excludes `bar`, it still detects
an infinite recursion, even though there is no reason it should process the
`bar` symlink. This makes it impossible to exclude problematic subpaths.

To replicate:

```
$ bazel build //:broken
INFO: Invocation ID: cd200d1d-5e86-4556-9408-e19cadbadcaf
ERROR: infinite symlink expansion detected
[start of symlink chain]
/home/user/bazel-glob-bug
/home/user/bazel-glob-bug/foo/bar
[end of symlink chain]
ERROR: Skipping '//:broken': no such package '': Symlink issue while evaluating globs: Infinite symlink expansion: /home/user/bazel-glob-bug/foo/bar- > /home/user/bazel-glob-bug
WARNING: Target pattern parsing failed.
ERROR: no such package '': Symlink issue while evaluating globs: Infinite symlink expansion: /home/user/bazel-glob-bug/foo/bar- > /home/user/bazel-glob-bug
INFO: Elapsed time: 0.071s
INFO: 0 processes.
FAILED: Build did NOT complete successfully (1 packages loaded)
```

Note that there is also a separate target that does not require or use this
glob expression at all: `//:working`. This just demonstrates that due to
the ordering of processing, any glob expression with this issue will
invalidate all neighboring build targets by interrupting evaluation of the
BUILD file.
