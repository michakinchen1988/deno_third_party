[![crates.io](https://img.shields.io/crates/v/dirs.svg)](https://crates.io/crates/dirs)
[![API documentation](https://docs.rs/dirs/badge.svg)](https://docs.rs/dirs/)
![actively developed](https://img.shields.io/badge/maintenance-actively--developed-brightgreen.svg)
[![TravisCI status](https://img.shields.io/travis/soc/dirs-rs/master.svg?label=Linux/macOS%20build)](https://travis-ci.org/soc/dirs-rs)
[![AppVeyor status](https://img.shields.io/appveyor/ci/soc/dirs-rs/master.svg?label=Windows%20build)](https://ci.appveyor.com/project/soc/dirs-rs/branch/master)
![License: MIT/Apache-2.0](https://img.shields.io/badge/license-MIT%2FApache--2.0-orange.svg)

# `dirs`

## Introduction

- a tiny low-level library with a minimal API
- that provides the platform-specific, user-accessible locations
- for retrieving and storing configuration, cache and other data
- on Linux, Windows (≥ Vista) and macOS.

The library provides the location of these directories by leveraging the mechanisms defined by
- the [XDG base directory](https://standards.freedesktop.org/basedir-spec/basedir-spec-latest.html) and
  the [XDG user directory](https://www.freedesktop.org/wiki/Software/xdg-user-dirs/) specifications on Linux
- the [Known Folder](https://msdn.microsoft.com/en-us/library/windows/desktop/dd378457.aspx) API on Windows
- the [Standard Directories](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html#//apple_ref/doc/uid/TP40010672-CH2-SW6)
  guidelines on macOS

## Platforms

This library is written in Rust, and supports Linux, macOS and Windows.

It's mid-level sister library, _directories_, is available for Rust ([directories-rs](https://github.com/soc/directories-rs))
and on the JVM ([directories-jvm](https://github.com/soc/directories-jvm)).

## Usage

#### Dependency

Add the library as a dependency to your project by inserting

```toml
dirs = "1.0"
```

into the `[dependencies]` section of your Cargo.toml file.

#### Example

Library run by user Alice:

```rust
extern crate dirs;

dirs::home_dir();
// Lin: Some(/home/alice)
// Win: Some(C:\Users\Alice)
// Mac: Some(/Users/Alice)

dirs::audio_dir();
// Lin: Some(/home/alice/Music)
// Win: Some(C:\Users\Alice\Music)
// Mac: Some(/Users/Alice/Music)

dirs::config_dir();
// Lin: Some(/home/alice/.config)
// Win: Some(C:\Users\Alice\AppData\Roaming)
// Mac: Some(/Users/Alice/Library/Preferences)

dirs::executable_dir();
// Lin: Some(/home/alice/.local/bin)
// Win: None
// Mac: None
```

## Design Goals

- The _dirs_ library is a low-level crate designed to provide the paths to standard directories
  as defined by operating systems rules or conventions. If your requirements are more complex,
  e. g. computing cache, config, etc. paths for specific applications or projects, consider using
  [directories](https://github.com/soc/directories-rs) instead.
- This library does not create directories or check for their existence. The library only provides
  information on what the path to a certain directory _should_ be. How this information is used is
  a decision that developers need to make based on the requirements of each individual application.
- This library is intentionally focused on providing information on user-writable directories only.
  There is no discernible benefit in returning a path that points to a user-level, writable
  directory on one operating system, but a system-level, read-only directory on another, that would
  outweigh the confusion and unexpected failures such an approach would cause.
  - `executable_dir` is specified to provide the path to a user-writable directory for binaries.<br/>
    As such a directory only commonly exists on Linux, it returns `None` on macOS and Windows.
  - `font_dir` is specified to provide the path to a user-writable directory for fonts.<br/>
    As such a directory only exists on Linux and macOS, it returns `None` on Windows.
  - `runtime_dir` is specified to provide the path to a directory for non-essential runtime data.
    It is required that this directory is created when the user logs in, is only accessible by the
    user itself, is deleted when the user logs out, and supports all filesystem features of the
    operating system.<br/>
    As such a directory only commonly exists on Linux, it returns `None` on macOS and Windows.

## Features

**If you want to compute the location of cache, config or data directories for your own application or project,
use `ProjectDirs` of the [directories](https://github.com/soc/directories-rs) project instead.**

| Function name    | Value on Linux                                                                                   | Value on Windows                  | Value on macOS                              |
| ---------------- | ------------------------------------------------------------------------------------------------ | --------------------------------- | ------------------------------------------- |
| `home_dir`       | `Some($HOME)`                                                                                    | `Some({FOLDERID_Profile})`        | `Some($HOME)`                               |
| `cache_dir`      | `Some($XDG_CACHE_HOME)`         or `Some($HOME`/.cache`)`                                        | `Some({FOLDERID_LocalAppData})`   | `Some($HOME`/Library/Caches`)`              |
| `config_dir`     | `Some($XDG_CONFIG_HOME)`        or `Some($HOME`/.config`)`                                       | `Some({FOLDERID_RoamingAppData})` | `Some($HOME`/Library/Preferences`)`         |
| `data_dir`       | `Some($XDG_DATA_HOME)`          or `Some($HOME`/.local/share`)`                                  | `Some({FOLDERID_RoamingAppData})` | `Some($HOME`/Library/Application Support`)` |
| `data_local_dir` | `Some($XDG_DATA_HOME)`          or `Some($HOME`/.local/share`)`                                  | `Some({FOLDERID_LocalAppData})`   | `Some($HOME`/Library/Application Support`)` |
| `executable_dir` | `Some($XDG_BIN_HOME`/../bin`)`  or `Some($XDG_DATA_HOME`/../bin`)` or `Some($HOME`/.local/bin`)` | `None`                            | `None`                                      |
| `runtime_dir`    | `Some($XDG_RUNTIME_DIR)`        or `None`                                                        | `None`                            | `None`                                      |
| `audio_dir`      | `Some(XDG_MUSIC_DIR)`           or `None`                                                        | `Some({FOLDERID_Music})`          | `Some($HOME`/Music/`)`                      |
| `desktop_dir`    | `Some(XDG_DESKTOP_DIR)`         or `None`                                                        | `Some({FOLDERID_Desktop})`        | `Some($HOME`/Desktop/`)`                    |
| `document_dir`   | `Some(XDG_DOCUMENTS_DIR)`       or `None`                                                        | `Some({FOLDERID_Documents})`      | `Some($HOME`/Documents/`)`                  |
| `download_dir`   | `Some(XDG_DOWNLOAD_DIR)`        or `None`                                                        | `Some({FOLDERID_Downloads})`      | `Some($HOME`/Downloads/`)`                  |
| `font_dir`       | `Some($XDG_DATA_HOME`/fonts/`)` or `Some($HOME`/.local/share/fonts/`)`                           | `None`                            | `Some($HOME`/Library/Fonts/`)`              |
| `picture_dir`    | `Some(XDG_PICTURES_DIR)`        or `None`                                                        | `Some({FOLDERID_Pictures})`       | `Some($HOME`/Pictures/`)`                   |
| `public_dir`     | `Some(XDG_PUBLICSHARE_DIR)`     or `None`                                                        | `Some({FOLDERID_Public})`         | `Some($HOME`/Public/`)`                     |
| `template_dir`   | `Some(XDG_TEMPLATES_DIR)`       or `None`                                                        | `Some({FOLDERID_Templates})`      | `None`                                      |
| `video_dir`      | `Some(XDG_VIDEOS_DIR)`          or `None`                                                        | `Some({FOLDERID_Videos})`         | `Some($HOME`/Movies/`)`                     |

## Comparison

There are other crates in the Rust ecosystem that try similar or related things.
Here is an overview of them, combined with ratings on properties that guided the design of this crate.

Please take this table with a grain of salt: a different crate might very well be more suitable for your specific use case.
(Of course _my_ crate achieves _my_ design goals better than other crates, which might have had different design goals.)

| Library                                                   | Status         | Lin | Mac | Win |Base|User|Proj|Conv|
| --------------------------------------------------------- | -------------- |:---:|:---:|:---:|:--:|:--:|:--:|:--:|
| [app_dirs](https://crates.io/crates/app_dirs)             | Unmaintained   |  ✔  |  ✔  |  ✔  | 🞈  | ✖  | ✔  | ✖  |
| [app_dirs2](https://crates.io/crates/app_dirs2)           | Maintained     |  ✔  |  ✔  |  ✔  | 🞈  | ✖  | ✔  | ✖  |
| **dirs**                                                  | **Developed**  |  ✔  |  ✔  |  ✔  | ✔  | ✔  | ✖  | ✔  |
| [directories](https://crates.io/crates/directories)       | Developed      |  ✔  |  ✔  |  ✔  | ✔  | ✔  | ✔  | ✔  |
| [s_app_dir](https://crates.io/crates/s_app_dir)           | Unmaintained?  |  ✔  |  ✖  |  🞈  | ✖  | ✖  | 🞈  | ✖  |
| [standard_paths](https://crates.io/crates/standard_paths) | Maintained     |  ✔  |  ✖  |  ✔  | ✔  | ✔  | ✔  | ✖  |
| [xdg](https://crates.io/crates/xdg)                       | Maintained     |  ✔  |  ✖  |  ✖  | ✔  | ✖  | ✔  | 🞈  |
| [xdg-basedir](https://crates.io/crates/xdg-basedir)       | Unmaintained?  |  ✔  |  ✖  |  ✖  | ✔   | ✖  | ✖  | 🞈  |
| [xdg-rs](https://crates.io/crates/xdg-rs)                 | Obsolete       |  ✔  |  ✖  |  ✖  | ✔   | ✖  | ✖  | 🞈  |

- Lin: Linux support
- Mac: macOS support
- Win: Windows support
- Base: Supports [generic base directories](https://github.com/soc/directories-rs#basedirs)
- User: Supports [user directories](https://github.com/soc/directories-rs#userdirs)
- Proj: Supports [project-specific base directories](https://github.com/soc/directories-rs#projectdirs)
- Conv: Follows naming conventions of the operating system it runs on

## Build

It's possible to cross-compile this library if the necessary toolchains are installed with rustup.
This is helpful to ensure a change hasn't broken code on a different platform.

The following commands will build this library on Linux, macOS and Windows:

```
cargo build --target=x86_64-unknown-linux-gnu
cargo build --target=x86_64-apple-darwin
cargo build --target=x86_64-pc-windows-gnu
```

## License

Licensed under either of

 * Apache License, Version 2.0
   ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license
   ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
