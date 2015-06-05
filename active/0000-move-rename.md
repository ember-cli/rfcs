- Start Date: 2015-06-04
- RFC PR: 
- ember-cli Issue: 

# Summary

Provide both an `ember move` and an `ember rename` command for ember-cli, providing the ability to move/rename a file as well as update any references to it in a project. The move command will be lower level, with the rename command being more user friendly. The commands will initially be developed as an addon and can later be brought into ember-cli.

# Motivation

Renaming through ember-cli is currently not possible, so if you mistype or need to change a name, it must be done manually, tracking down all assets generated with a blueprint. Alternatively, you can use the `ember d` command, then regenerate with `ember g`, but this is not desirable when you've already got content in your generated file(s). Additionally, if you've already made references to the module you want to change (like a component for instance), you have to search and replace through your entire project to ensure you don't leave any orphaned references.

# Detailed design

The `ember move` command and the `ember rename` commands will essentially do the same thing, with the difference being `ember rename` will be built on top of `ember move` and will take a blueprint as an argument (which it will use to help generate the paths passed to `ember move`). The rename command will also find all associated files that are generated with the defined blueprint and move them (and update path references in the project) as well.

## Command: ember move
While the `ember move` command will essentially be a proxy for `git mv`, the more important role it will serve is updating any references to the source path inside a project. User prompting with diffs should be done with any change, bypassable with the `--force` option. If the file changes folder depth, any import statements with relative paths will also be updated.

It will be used by the `ember rename` command, as well as by future migration tools in ember-watson. The aim is to keep it as basic as possible to allow building upon it with other commands.

If the file is versioned, use `git mv` to keep the version history (check via `git ls-files --error-unmatch <file_name>; echo $?`). If not, use `mv` or `move`.

### Usage
ember move [mv]: `ember mv <source> <dest> [options]...`

The source and dest arguments should be paths to files, directories will be ignored. Paths should either be relative to the executing path, or start with a `/` to start from the project root.

#### Arguments
| name | description |
| ---- | ----------- |
| source | path to file from current dir or project root |
| dest | path to destination from current dir or project root |

#### Options
| name | description |
| ---- | ----------- |
| dry-run | run and log output without actually executing |
| verbose | log all actions |
| force | overwrite any existing destination files |

### Examples:
Given the following app structure:
```
app
├── components
│   └── foo-bar.js
│   └── bar
│       └── foo-baz.js
├── routes
...
```
```
// using project root paths
ember mv /app/components/foo-bar.js /app/components/bar-foo.js

// this will work if run if foo-bar.js exists in the executing path
ember mv foo-bar.js bar-foo.js

// relative path examples
ember mv foo-bar.js foo/bar-foo.js
ember mv ../../foo.js foo.js
ember mv ./foo.js bar.js

// moving from relative path to root path
ember mv foo-bar.js /app/components/bar/foo-bar.js
```
 
### Process

1. `ember mv <source> <dest> [options]...`
2. pre-process:
  1. pre-verify:
    * SOURCE file exists
    * DEST file does not already exist
  2. check for git
    * check for .git in project
    * check to see if file is versioned
  3. check dest dir exists
  4. beforeMove hook
  5. prompt user
3. process:
  1. create dest dir(s) if they don't already exist
  2. `[git] [mv,move] <source> <dest>`
4. post-process:
  1. search project for import paths referencing source path (relative paths as well)
    * get AST for files matching path
    * update instances of source path with dest path
  2. afterMove hook

## Command: ember rename
The `ember rename` is similar to the `ember mv` command, but instead of paths, takes an additional `blueprint` argument that provides context to generate the source and destination paths. The `ember rename` command leverages the `ember mv` command, generating paths for the source and dest, then calling the command. The rename command will update all files that would normally be generated from a blueprint.

### Usage
ember rename [r]: `ember r <blueprint> <source> <dest> [options]...`

The arguments; blueprint, source, and dest should be strings with no extension, as it is derived from the files associated with the defined blueprint argument.

#### Arguments
| name | description |
| ---- | ----------- |
| blueprint | name of the blueprint to use as context |
| source | name of existing generated blueprint |
| dest | path to generate destination |

#### Options
| name | description |
| ---- | ----------- |
| dry-run | run and log output without actually executing |
| verbose | log all actions |
| force | overwrite any existing destination files |
| structure | structure to use as context for looking up path (pod, type, addon) |

### Examples:
Given the following app structure:
```
app
├── components
│   └── foo-bar.js
│   └── bar
│       └── foo-baz.js
├── routes
├── templates
│   └── components
│       └── foo-bar.js
│       └── bar
│           └── foo-baz.js
...
```
```
ember r component foo-bar bar-foo
```
 
### Process

1. `ember r <blueprint> <source> <dest> [options]...`
2. pre-process:
  1. generate paths
    * lookup BLUEPRINT
    * get fileInfos for any files that would be generated
    * construct paths for source and dest
  2. pre-verify:
    * SOURCE path exists 
    * DEST path does not already exist
  4. beforeMove hook
  5. prompt user
3. process:
  1. create dest dir(s) they don't already exist
  2. loop through file list and run `ember mv <source> <dest> <options>`
4. post-process:
  1. afterMove hook
  
# Out of scope

Renaming entire directories with `ember mv` would be a nice to have, specifically for the capability to update path references in a project. It will likely need to wait for a simpler, solid version of the command.

To reduce scope we will likely require the project to be versioned and only use `git mv`. Instead of falling back to `mv` or `move`, we will throw a warning that git is required for the command. Eventually a fallback can be provided, but for the initial pass it will not.

# Drawbacks

Because the mv command has a long established history, the limited functionality of this implementation could cause confusion for many users. There also may be many corner cases that won't immediately be covered by the command, possibly breaking apps, causing more issues than it solves. As long as the command leaves a trail and log of what it did, undoing any accidental changes should be managable.

# Alternatives

Ember-cli alternatives are unknown, the only way to do this currently is manually through the command line and with a text editor with search and replace capabilities.

# Unresolved questions

none so far