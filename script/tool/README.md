# Flutter Plugin Tools

This is a set of utilities used in this repository, both for CI and for
local development.

The tool is designed to be run at the root of the repository or `<repository-root>/packages/`.

## Getting Started

In flutter/packages, the tool is run from source.

Set up:

```sh
cd script/tool && dart pub get && cd ../../
```

Run:

```sh
dart run script/tool/bin/flutter_plugin_tools.dart <args>
```

Many commands require the Flutter-bundled version of Dart to be the first `dart` in the path.

### Extra Setup

When updating sample code excerpts (`update-excerpts`) for the README.md files,
there is some [extra setup for
submodules](#update-readmemd-from-example-sources) that is necessary.

## Commands

Run with `--help` for a full list of commands and arguments, but the
following shows a number of common commands being run for a specific package.

Most commands take a `--packages` argument to control which package(s) the
command is targetting. An package name can be any of:
- The name of a package (e.g., `path_provider_android`).
- The name of a federated plugin (e.g., `path_provider`), in which case all
  packages that make up that plugin will be targetted.
- A combination federated_plugin_name/package_name (e.g.,
  `path_provider/path_provider` for the app-facing package).

### Format Code

```sh
cd <repository root>
dart run script/tool/bin/flutter_plugin_tools.dart format --packages package_name
```

### Run the Dart Static Analyzer

```sh
cd <repository root>
dart run script/tool/bin/flutter_plugin_tools.dart analyze --packages package_name
```

### Run Dart Unit Tests

```sh
cd <repository root>
dart run script/tool/bin/flutter_plugin_tools.dart test --packages package_name
```

### Run Dart Integration Tests

```sh
cd <repository root>
dart run script/tool/bin/flutter_plugin_tools.dart build-examples --apk --packages package_name
dart run script/tool/bin/flutter_plugin_tools.dart drive-examples --android --packages package_name
```

Replace `--apk`/`--android` with the platform you want to test against
(omit it to get a list of valid options).

### Run Native Tests

`native-test` takes one or more platform flags to run tests for. By default it
runs both unit tests and (on platforms that support it) integration tests, but
`--no-unit` or `--no-integration` can be used to run just one type.

Examples:

```sh
cd <repository root>
# Run just unit tests for iOS and Android:
dart run script/tool/bin/flutter_plugin_tools.dart native-test --ios --android --no-integration --packages package_name
# Run all tests for macOS:
dart run script/tool/bin/flutter_plugin_tools.dart native-test --macos --packages package_name
# Run all tests for Windows:
dart run script/tool/bin/flutter_plugin_tools.dart native-test --windows --packages package_name
```

### Update README.md from Example Sources

`update-excerpts` requires sources that are in a submodule. If you didn't clone
with submodules, you will need to `git submodule update --init --recursive`
before running this command.

```sh
cd <repository root>
dart run script/tool/bin/flutter_plugin_tools.dart update-excerpts --packages package_name
```

### Update CHANGELOG and Version

`update-release-info` will automatically update the version and `CHANGELOG.md`
following standard repository style and practice. It can be used for
single-package updates to handle the details of getting the `CHANGELOG.md`
format correct, but is especially useful for bulk updates across multiple packages.

For instance, if you add a new analysis option that requires production
code changes across many packages:

```sh
cd <repository root>
dart run script/tool/bin/flutter_plugin_tools.dart update-release-info \
  --version=minimal \
  --base-branch=upstream/main \
  --changelog="Fixes violations of new analysis option some_new_option."
```

The `minimal` option for `--version` will skip unchanged packages, and treat
each changed package as either `bugfix` or `next` depending on the files that
have changed in that package, so it is often the best choice for a bulk change.

For cases where you know the change type, `minor` or `bugfix` will make the
corresponding version bump, or `next` will update only `CHANGELOG.md` without
changing the version.

If you have a standard repository setup, `--base-branch=upstream/main` will
usually give the behavior you want, finding all packages changed relative to
the branch point from `upstream/main`. For more complex use cases where you want
a different diff point, you can pass a different `--base-branch`, or use
`--base-sha` to pick the exact diff point.

### Update a dependency

`update-dependency` will update a pub dependency to a new version.

For instance, to updated to version 3.0.0 of `some_package` in every package
that depends on it:

```sh
cd <repository root>
dart run script/tool/bin/flutter_plugin_tools.dart update-dependency \
  --pub-package=some_package \
  --version=3.0.0 \
```

If a `--version` is not provided, the latest version from pub will be used.

Currently this only updates the dependency itself in pubspec.yaml, but in the
future this will also update any generated code for packages that use code
generation (e.g., regenerating mocks when updating `mockito`).

### Publish a Release

**Releases are automated for `flutter/packages`.**

The manual procedure described here is _deprecated_, and should only be used when
the automated process fails. Please, read
[Releasing a Plugin or Package](https://github.com/flutter/flutter/wiki/Releasing-a-Plugin-or-Package)
on the Flutter Wiki first.

```sh
cd <path_to_repo>
git checkout <commit_hash_to_publish>
dart run script/tool/bin/flutter_plugin_tools.dart publish --packages <package>
```

By default the tool tries to push tags to the `upstream` remote, but some
additional settings can be configured. Run `dart run script/tool/bin/flutter_plugin_tools.dart
publish --help` for more usage information.

The tool wraps `pub publish` for pushing the package to pub, and then will
automatically use git to try to create and push tags. It has some additional
safety checking around `pub publish` too. By default `pub publish` publishes
_everything_, including untracked or uncommitted files in version control.
`publish` will first check the status of the local
directory and refuse to publish if there are any mismatched files with version
control present.

## Updating the Tool

For flutter/packages, just changing the source here is all that's needed.

For changes that are relevant to flutter/packages, you will also need to:
- Update the tool's pubspec.yaml and CHANGELOG
- Publish the tool
- Update the pinned version in
  [flutter/packages](https://github.com/flutter/packages/blob/main/.cirrus.yml)