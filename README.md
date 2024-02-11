Bug in helm-3.14 subkey removal

We have "subchart" and it's dependency called "subsubchart". Both subcharts have the following values.yaml files:
```
hash:
  key1: 1
  key2: 2

  subhash:
    key1: a
    key2: b
```

We want to try and remove hash.key1 and hash.subhash.key1 in both "subchart" and "subsubchart". We do so with the following top-level values.yaml file:
```
subchart:
  hash:
    key1: null # Expect this to be removed in subchart
    key2: 333
    subhash:
      key1: null # Expect this to be removed in subchart
      key2: test

  subsubchart:
    hash:
      key1: null # Expect this to be removed in subsubchart
      key2: thisworks
      subhash:
        key1: null # Expect this to be removed in subsubchart
        key2: overrideworks
```

When templating this we expect that key1 is removed from both subchart and subsubchart.

```
$ helm-3.14.0 version
version.BuildInfo{Version:"v3.14.0", GitCommit:"3fc9f4b2638e76f26739cd77c7017139be81d0ea", GitTreeState:"clean", GoVersion:"go1.21.5"}

$ helm-3.14.0 template .
---
# Source: demo/charts/subchart/charts/subsubchart/templates/subsubchart-test.yaml
global: {}
hash:
  key1: 1 # Not removed in subsubchart
  key2: thisworks
  subhash:
    key1: a # Not removed in subsubchart
    key2: overrideworks
---
# Source: demo/charts/subchart/templates/subchart-test.yaml
global: {}
hash:
  key2: 333 # (Note: key1 is correctly removed above)
  subhash:
    key2: test # (Note: key1 is correctly removed above)
subsubchart:
  global: {}
  hash:
    key1: 1
    key2: thisworks
    subhash:
      key1: a
      key2: overrideworks
```

We can observe the following from the output above:
1. The key removal works fine in "subchart" (both under "hash" and under "hash.subhash")
2. The key removal is broken in "subsubchart" and both "key1" entries are *not* removed as expected.

The same happens with helm-3.13.*
