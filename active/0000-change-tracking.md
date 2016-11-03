- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)
- Ember CLI Issue: (leave this empty)

# Summary

Migrate broccoli plugins from input diffing, to change tracking.

* abstract input trees, output trees, change calculation

# Motivation

* plugin author ergonomics: existing "pit of sucess" when authoring broccoli plugins performantly isn't their yet (but can be)
* build performance: in most apps, rebuild times are dominated by the accurate diff based change detection
* symlinks are not friendly cross platform
* accessive disk writes caused by symlinks is costly both in terms of time and interms of disk lifetimes
* accessive files and directories on disk, cause other tooling to have issues

Experimation at: https://github.com/stefanpenner/fs-tree-diff/pull/36

# Detailed design


### Basic "single file compiler" example:

```js
build() {
  // read from out input

  let content = this.in.readFileSync('my-input-file.js', 'UTF8');

  let  { map, code } = compile(content);

  // write to our output
  this.out.writeFileSync('out.js',  code);
  this.out.writeFileSync('out.map', map);
}
```

##### This enables:

* hide `this.inputTree[0] + my-input-file.js` related string manipulation
* hide `this.output + 'out.js'` related string manipulation
* automatically (with opt-out) only write, of the contents checksum has changed (keeps builds stable)
* hides actual location on disk (removal of symlinks, while keeping memory pressure low by keeping contents on disk, but only when they change)
* automatically track changes (by tracking all calls to `this.out.*`
* disable writes to your input (fail fast)


### broccoli-filter'esq "apply only changes" API

```js
build() {
  this.in.changes().forEach(change => {
    let [
      operation,
      relativePath,
      entry
    ] = change; // TODO: should we change/expand this structure

    // TODO: this is a common pattern, a subsequent (higher level RFC should aim to explore more ergonomic approaches)
    switch(operation) {
      case 'create':;
      case 'change': return this.out.writeFileSync(relativePath, this.compile(relativePath, this.in.readFileSync(relativePath)));
      case 'unlink': return this.out.unlinkSync(relativePath);
      case 'rmdir' : return this.out.rmdirSync(relativePath);
      case 'mkdir' : return this.out.mkdir(relativePath);
    }
  })l
}
```

##### This enables:

* everything from the above
* `changes()` do not need to be calculated, rather they are the changes tracked
  from the input plugin's `this.out.*` calls, if there was no upstream plugin,
  or it does not yet use the change tracking semantics, `this.in` automatically
  calculates the changes (or maybe recieved it directly from watchman or similar).
* `this.out.*` calls are automatically tracked for downstream plugins `changes`
* `this.out.*` calls are automatically skipped if idempotent (writing the same
  file contents twice);


### `this.in.glob`

#### signature

* pojo, which optionals contains `.includes` and `.excludes` which are both
  optional but if present must be arrays of zero or more globs

A common use-case is to want reduce the number of files one plugin operates on.
For example, we wish to only operate on `app/components/**/*`

```js
this.in.glob({ includes: ['app/components/**/*']); // scoped `fs facade` with all the same functions just filtered to only  files from `app/components/`
```

#### This enables

* built-in globbing and walkSync
* reducing the scope of `changes` we listen to, if the underlying change watching system can be reduces
* nearly implements all of `broccoli-filter` but as a primitive of the system

### Multiple Input trees

This is where the system has the potentially to really shine. In the vast
majority of cases, when recieving multiple input trees we ultimately `merge`
them via `mergeTrees` and then consume the resulting tree. This means, we must
always merge even if the subsequent plugin only requires a small subset of the
resulting merge. So instead of eagerly merging, and writing the merge to disk.
We can treat each merge input as a loadPath, and only read what we need without
incuring the overhead of an eager merge.

##### Example:

```
tree 1: /a/{a,b,c}.txt
tree 2: /a/{a,b,d}.txt
```

and tree 3 was implemented as:

```
new Funnel([tree1, tree2], {
  files: ['a/c.txt', a/a.txt'']
})
```

Rather then:

1. merge 1 + 2 into a new directory with many dir and symlinks
2. then glob/walkSync the new tree of symlnks looking for `a/c.txt`

We can:

1. search for `a/c.txt` in [tree1, tree2] (search right to left, like merge-trees)
2. it isn't in tree2 at position `a/c.txt`, look back one
3. it is in tree1 at position `a/c.txt`
4. return tree1's `a/c.txt`

TODO: more indepth algorithim and examples
TODO: think above and explore globbing across loadpaths
TODO: should we keep aroud mergeTrees, as an public API but internally its
implemented as described. Should we do so with the intent to deprecate, and let
every plugin take in arrays?

#### This enables

* no symlinks
* no on-disk representation of merges (unless we want to)
* no speculative merging, only to pluck out a subset of files
* propogate changes through merges
* merge as a primitive of the platform (merge-trees becomes largely a noop);

TODO: list entire proposed API surface
TODO: describe the single entries data-structure that contains both the stat information and true path on disk, this may also want to (if possible) track the original input source on disk
TODO: do we still create the existing default broccoli plugin tmp directories, or do one of this plugins opt-out

### Transition Plan

To ease upgrade transitions, and allow us to incrementally learn and experiment
the idea is for `this.in` to fallback to legacy diffing of its input, if the
input is a legacy plugin.

TODO: how exactly do we detect non-legacy plugins (likely some part of broccolis capabilities stuff)

# How We Teach This

TODO

# Drawbacks

More magic, extra indirection.

# Alternatives

* Switch to different tool
* hand roll optimizations
* provide a series of high level libs that together could create this experience.
* What other designs have been considered? What is the impact of not doing this?

# Unresolved questions

* at the very least, quite abit of implementation questions
* TODO
