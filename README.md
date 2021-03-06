# Rocket 🚀

Swift CLI release tool for Git repos and Swift Packages

## Installation
You can install Rocket with SPM

1. Add or amend a Package.swift
2. Add this dependency `.package(url: "https://github.com/f-meloni/Rocket", from: "0.1.0")`
3. Then you can run `swift run rocket 1.0.0` where `1.0.0` is the version that you want to release

## Setup the release steps

#### With YAML
Create a file called `.rocket.yml` with your steps inside

```yaml
---
steps: 
  - script: 
      content: ruby Scripts/update_changelog.rb
  - git_add:
      paths:
        - CHANGELOG.md
  - commit:
      message: "Releasing version $VERSION"
  - tag
  - push
```

#### With PackageConfig
With PackageConfig (https://github.com/orta/PackageConfig) you can put the configuration at the end of your `Package.swift`

```swift
#if canImport(PackageConfig)
    import PackageConfig
    
    let config = PackageConfig([
        "rocket": ["steps":
            [
                ["script": ["content": "ruby Scripts/update_changelog.rb"]]
                ["git_add": ["paths": ["CHANGELOG.md"]]],
                ["commit": ["message": "Releasing version $VERSION"]],
                "tag",
                "push"
            ]
        ]
    ])
```

## Before/After steps
If you use the `before` and/or `after` keys

e.g.

```yaml
---
before: 
  - script: 
      content: echo "Testing Release for $VERSION"
after:
  - script: 
      content: echo "released $VERSION"
```

Rocket will execute some default steps between the before and after steps:

- `echo "Testing Release for $VERSION"`

- **hide_dev_dependencies**
- **commit**
- **unhide_dev_dependencies**
- **commit** (message: "Unhide dev dependencies")

- `echo "released $VERSION"`

## Hide dev dependencies
When you release a package you want that who adds it as dependency downloads just the dependencies that are really needed to your package.

This is why Rocket introduces the concept of dev dependency, if you have in your package some scripts e.g. swiftformat you can add them as dev dependencies (by adding `// dev` after them) and they will be commented by the `hide_dev_dependencies` step and uncommented from the `unhide_dev_dependencies`.

That is also valid for the dependencies that are used just from test targets, but in that case you will have to add the test target as dev dependency too.<br/><br/>

Some examples are:
- https://github.com/danger/swift/blob/master/Package.swift
- https://github.com/f-meloni/Rocket/blob/master/Package.swift
- https://github.com/f-meloni/Logger/blob/master/Package.swift

## Supported Steps

### script
Runs a command line script

**Parameters:**

_content_: `String` (required): the script content
<br/><br/>

### git_add
Adds the files to the git's staging area

**Parameters:**

_paths_: `[String]` (optional): The paths you want to add to the staging area - **default**: `[.]`
<br/><br/>

### commit
Commits on git

**Parameters:**

_message_: `String` (optional): The commit message - **default**: `"Version $VERSION"`

_no_verify_: `Bool` (optional): bypasses the pre-commit and commit-msg hooks **default**: `false`
<br/><br/>

### tag
Tags the current version
<br/><br/>

### push
Pushes the current changes

**Parameters:**

_remote_: `String` (optional): The name of the remote you want to push to - **default**: `"origin"`

_branch_: `String` (optional): The name of the branch you want to push - **default**: `"master"`
<br/><br/>

### hide_dev_dependencies
Comment the dev dependencies on your `Package.swift` to avoid them to be shipped with your release
Dev dependencies are all the lines in the `Package.swift` file with `// dev`

**Parameters:**

_package_path_: `String` (optional): The relative path to your `Package.swift` file - **default**: `"Package.swift"`
<br/><br/>

### unhide_dev_dependencies
Uncomment the dev dependencies on your `Package.swift` after you released.

**Parameters:**

_package_path_: `String` (optional): The relative path to your `Package.swift` file - **default**: `"Package.swift"`
<br/><br/>

### swift_script
Compile and run a script file written in swift

The version is passed to the script as first parameter

**Parameters:**

_script_path_: `String` (required): The relative path to the swift script

_arguments_: `[String]` (optional): Additional arguments for the swift script
<br/><br/>

## Variable $VERSION

You can use the variable `$VERSION` inside the steps to refer to the version you are releasing

## Next steps

- [X] Add a step to execute script files written in swift
- [X] Add a step to comment the dev dependencies on the `Package.swift`
- [ ] Add a step to create Github releases
