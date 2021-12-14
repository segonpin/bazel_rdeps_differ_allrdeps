# Show inconsistent behavior with rdeps and allrdeps

Having three targets with the dependency graph `//:A -> //subdirectory:B -> //:C`, we obtain different behaviors from rdeps and allrdeps even if we specify the same universe.

Doing normal query
```
$ bazelisk query --universe_scope=//...  --order_output=no "allrdeps(//:C)"
//:C
//subdirectory:B
//:A

```
and doing a SkyQuery
```
$ bazelisk query "rdeps(//..., //:C)"
//:A
//subdirectory:B
//:C
```

After renaming/removing the BUILD file in `subdirectory`, that hides `//subdirectory:B`, 
```
$ mv subdirectory/BUILD subdirectory/hidden.BUILD
```
doing normal query
```
$ bazelisk query --universe_scope=//...  --order_output=no "allrdeps(//:C)"
//:C
```
and doing a SkyQuery
```
$ bazelisk query "rdeps(//..., //:C)"
ERROR: /home/sebas/src/bazelbuild/bazel_rdeps_differ_allrdeps/BUILD:3:11: no such target '//subdirectory:B': target 'B' not declared in package 'subdirectory' defined by /home/sebas/src/bazelbuild/bazel_rdeps_differ_allrdeps/subdirectory/BUILD and referenced by '//:A'
ERROR: /home/sebas/src/bazelbuild/bazel_rdeps_differ_allrdeps/BUILD:3:11: no such target '//subdirectory:B': target 'B' not declared in package 'subdirectory' defined by /home/sebas/src/bazelbuild/bazel_rdeps_differ_allrdeps/subdirectory/BUILD and referenced by '//:A'
ERROR: Evaluation of query "rdeps(//..., //:C)" failed: errors were encountered while computing transitive closure
Loading: 0 packages loaded
```
Then "un-hide" file again to get clean git status
```
$ mv subdirectory/hidden.BUILD subdirectory/BUILD
```

Is that expected?