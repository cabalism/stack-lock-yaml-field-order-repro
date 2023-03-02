# lock-yaml-field-order

On noticing a field order change in `stack.yaml.lock` I added one source
repository package extra dependency to a stack project generated using:

```
stack new lock-yaml-field-order new-template --bare
```

```diff
diff --git a/stack.yaml b/stack.yaml
index c39389f..6e7ee06 100644
--- a/stack.yaml
+++ b/stack.yaml
@@ -1,3 +1,6 @@
 resolver: lts-20.12
 packages:
-- .
\ No newline at end of file
+- .
+extra-deps:
+  - git: git@github.com:bitnomial/prometheus.git
+    commit: 1fde661d897c500534769960584b29e8c39e8abb
```

Using ghcup, I regenerated the lock file implicitly (explicitly with
`--stack-yaml=stack.yaml` yields the same results).

With stack versions 2.7.3 and 2.7.5 the field order is:

```yaml
packages:
- completed:
    name: prometheus
    version: 2.2.2
    git: git@github.com:bitnomial/prometheus.git
    pantry-tree:
      size: 1969
      sha256: 904843022ed0eabaf4e3dfc8f36e28e7365788c91e1d471b1eba56e1fca7511d
    commit: 1fde661d897c500534769960584b29e8c39e8abb
  original:
    git: git@github.com:bitnomial/prometheus.git
    commit: 1fde661d897c500534769960584b29e8c39e8abb
snapshots:
- completed:
    size: 649133
    url: https://raw.githubusercontent.com/commercialhaskell/stackage-snapshots/master/lts/20/12.yaml
    sha256: af5d667f6096e535b9c725a72cffe0f6c060e0568d9f9eeda04caee70d0d9d2d
  original: lts-20.12
```

With stack versions 2.9.1 and 2.9.3 the field order is:

```yaml
packages:
- completed:
    commit: 1fde661d897c500534769960584b29e8c39e8abb
    git: git@github.com:bitnomial/prometheus.git
    name: prometheus
    pantry-tree:
      sha256: 904843022ed0eabaf4e3dfc8f36e28e7365788c91e1d471b1eba56e1fca7511d
      size: 1969
    version: 2.2.2
  original:
    commit: 1fde661d897c500534769960584b29e8c39e8abb
    git: git@github.com:bitnomial/prometheus.git
snapshots:
- completed:
    sha256: af5d667f6096e535b9c725a72cffe0f6c060e0568d9f9eeda04caee70d0d9d2d
    size: 649133
    url: https://raw.githubusercontent.com/commercialhaskell/stackage-snapshots/master/lts/20/12.yaml
  original: lts-20.12
```

If storing the lock file in source control this will result in large git diffs
but not semantic diffs if using a YAML-aware diff tool such as
[graphtage][graphtage].

```diff
git diff --no-index stack.implicit-v275.lock stack.implicit-v291.lock
diff --git a/stack.implicit-v275.lock b/stack.implicit-v291.lock
index c1313fc..02ea2fe 100644
--- a/stack.implicit-v275.lock
+++ b/stack.implicit-v291.lock
@@ -5,19 +5,19 @@
 
 packages:
 - completed:
-    name: prometheus
-    version: 2.2.2
+    commit: 1fde661d897c500534769960584b29e8c39e8abb
     git: git@github.com:bitnomial/prometheus.git
+    name: prometheus
     pantry-tree:
-      size: 1969
       sha256: 904843022ed0eabaf4e3dfc8f36e28e7365788c91e1d471b1eba56e1fca7511d
-    commit: 1fde661d897c500534769960584b29e8c39e8abb
+      size: 1969
+    version: 2.2.2
   original:
-    git: git@github.com:bitnomial/prometheus.git
     commit: 1fde661d897c500534769960584b29e8c39e8abb
+    git: git@github.com:bitnomial/prometheus.git
 snapshots:
 - completed:
+    sha256: af5d667f6096e535b9c725a72cffe0f6c060e0568d9f9eeda04caee70d0d9d2d
     size: 649133
     url: https://raw.githubusercontent.com/commercialhaskell/stackage-snapshots/master/lts/20/12.yaml
-    sha256: af5d667f6096e535b9c725a72cffe0f6c060e0568d9f9eeda04caee70d0d9d2d
   original: lts-20.12
```

The commands I used for this test, run like this:

```
$ ghcup set stack 2.7.5
$ rm stack.yaml.lock
$ stack build --dry-run
$ cp stack.yaml.lock stack.implicit-v275.lock
```

```
$ ghcup set stack 2.9.1
$ rm stack.yaml.lock
$ stack build --dry-run
$ cp stack.yaml.lock stack.implicit-v291.lock
```

[graphtage]: https://github.com/trailofbits/graphtage