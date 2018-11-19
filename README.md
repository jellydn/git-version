# git-version

Git versioning used in Codacy.

The goal is to have a simple versioning system for our internal projects that we can use to track the different releases.

This tool returns in standard output the current version (e.g. to be used for tagging) depending on the git history and commit messages.

The versioning scheme is assumed to be __Semver__.

## Requirements

To use this tool you will need to install a few dependencies:

Ubuntu:
```
sudo apt-get install \
  libevent-dev \
  git
```

Fedora:
```
sudo dnf -y install \
  libevent-devel \
  git
```

Alpine:
```
apk add --update --no-cache --force-overwrite \
  gc-dev pcre-dev libevent-dev \
  git
```

OsX:
```
brew install \
  libevent \
  git
```

## Versioning Model

Creates a version with the format `MAJOR.MINOR.PATCH`

**To use this you need to be in the working dir of a git project:**
```
$ ./git-version
1.0.0
```

Versions are incremented since last tag. The patch version is incremented by default, unless there is at least one commit since the last tag, containing `feature:` or `breaking:` in the message.

On branches other than master and `dev` the version is a variation of the latest common tag with master, and has the following format:

`{MAJOR}.{MINOR}.{PATCH}-{sanitized-branch-name}.{hash}`

On the `dev` branch the format is following:

`{MAJOR}.{MINOR}.{PATCH}-SNAPSHOT.{hash}`

*Example:*
```
---A---B---C <= Master (tag: 1.0.1)        L <= Master (git-version: 1.0.2)
            \                             /
             D---E---F---G---H---I---J---K <= Foo (git-version: 1.0.2-foo.5e30d83)
```


*Example2 (with dev branch):*
```
---A---B---C <= Master (tag: 1.0.1)        L <= Master (git-version: 1.0.2)
            \                             / <= Fast-forward merges to master (same commit id)
             C                           L <= Dev (git-version: 1.0.2-SNAPSHOT.5e30d83)
              \                         /
               E---F---G---H---I---J---K <= Foo (new_version: 1.0.1-foo.5e30d83)
```

*Example3 (with breaking message):*
```
---A---B---C <= Master (tag: 1.0.1)        L <= Master (git-version: 2.0.0)
            \                             /
             D---E---F---G---H---I---J---K <= Foo (git-version: 2.0.0-foo.5e30d83)
                                         \\
                                         message: "breaking: removed api parameter"
```

## CircleCI

Use this image directly on CircleCI for simple steps

```
version: 2
jobs:
  build:
    machine: true
    working_directory: /app
    steps:
      - checkout
      - run:
          name: get new version
          command: |
            NEW_VERSION=$(docker run --rm -v $(pwd):/repo codacy/git-version)
            echo $NEW_VERSION
```

## Build and Publish

The pipeline in `circleci` can deploy this for you when the code is pushed to the remote.

To compile locally you need to install [crystal](https://crystal-lang.org/docs/installation/) and possibly [all required libraries](https://github.com/crystal-lang/crystal/wiki/All-required-libraries)

You can also run everything locally using the makefile.

To get the list of available commands:
```
$ make help
```

### Credits

Great inspiration for this tool has been taken from: [GitVersion](https://github.com/GitTools/GitVersion)