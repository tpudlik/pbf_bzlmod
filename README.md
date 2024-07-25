# Platform-based flags and bzlmod

This repository demonstrates an unfortunate interaction between platform-based
flags and bzlmod.

Repro steps:

1.  `cd` into `downstream`.
1.  Run `bazelisk build //:library`. This "just works".
1.  Now run `bazelisk build --@upstream//:label_flag=@upstream//:alt
    //:library`. This also "just works": `//:library` depends on
    `@upstream//:label_flag`, which we set to a new value.
1.  Now, try `bazelisk build --platforms=:alt_platform //:library`. The platform
    `:alt_platform` has the attribute,

    ```
    flags = [
        "--@upstream//:label_flag=@upstream//:alt",
    ],
    ```

    So, the expected result is the same as in step 3. Instead, I get the error,

    ```
    $ bazelisk build --platforms=:alt_platform //:library
    WARNING: Build options --@@upstream~//:label_flag and --platforms have changed, discarding analysis cache (this can be expensive, see https://bazel.build/advanced/performance/iteration-speed).
    ERROR: no such package '@@upstream//': The repository '@@upstream' could not be resolved: Repository '@@upstream' is not defined
    ERROR: /usr/local/google/home/tpudlik/.cache/bazel/_bazel_tpudlik/70950bf1321433197772d904dffeb6b1/external/upstream~/BUILD.bazel:3:11: every rule of type label_flag implicitly depends upon the target '@@upstream//:alt', but this target could not be found because of: no such package '@@upstream//': The repository '@@upstream' could not be resolved: Repository '@@upstream' is not defined
    ```

I can fix this by changing the `flags` attribute to,

```
flags = [
    "--@upstream//:label_flag=@@upstream~//:alt",
],
```

But this is very unintuitive, and seems to depend on some bzlmod implementation
detail.

Note: the `.bazelversion` used in the `downstream` project was chosen to contain
the fix for https://github.com/bazelbuild/bazel/issues/22995.
