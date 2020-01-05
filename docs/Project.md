# Project

A project is a bundle of files in a folder:

  1. A configuration file describing the project.
  1. Source code in a file or sub-folder.
  1. Optionally other files.

A project name must be:

  1. a valid APL name.
  1. consist of nothing but ASCII (**not** ANSI) characters.

## 1. Config file

The config file is a JSON file with the name `apl-project.json` and is defined by the [apl-project-schema](./apl-project-schema.json). Below is an example file for a package.

```json
{
  "group": "GroupA",
  "name": "PackageA",
  "version": "1.0.0",
  "source": "src",
  "dependencies": [
    {
      "group": "GroupB",
      "name": "PackageB",
      "version": "0.9.10",
    },
    {
      "group": "GroupC",
      "name": "PackageC",
      "version": "4.3.9",
      "alias": "C4"
    },
    {
      "group": "GroupC",
      "name": "PackageC",
      "version": "5.0.0",
      "alias": "C5",
      "inject": "all"
    }
  ]
}
```

### 1.1. Source code

The source code could be a single file or folder. The format of the source file(s) must be supported by `Link`.

## 1.2. Project ID

A project is uniquely identified by its group and name. The ID string for a project is formed by combining group and name with a dash (ie. `{group}-{name}`). For packages, the version number is appended to the end with a prefix of `v` (`{group}-{name}-v{version}`), eg. `Dyalog-Conga-v3.1.3`.