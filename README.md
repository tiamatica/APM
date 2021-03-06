# APL Package Manager - Mark II

    Authors: Gilgamesh Athoraya, Kai Jäger

## Summary

This document aims to define what an APL Package is and a manager to deal with such packages. In its simplest form an implementation can work without a server process and simply deal with packages directly without a supporting registry. The complete solution allows interacting with both local registries and served registries (APL Package server).

## 1. The Package

A package is a bundle of files in a folder:
  1. A configuration file describing the package. 
  1. Source code in a file or sub-folder
  1. Optionally other files (resources)

A package name must be:
  1. a valid APL name
  1. consist of nothing but ASCII (**not** ANSI) characters

A package folder is named after the package group, name and version as defined in the config file (eg. `MyGroup-MyPackage-v1.0.0`).

### 1.1. Config file

The config file is a JSON file with the name `apl-project.json`. It defines the following properties:

  * name
  * group
  * version
  * source (code source path, single file or folder)
  * files (whitelist of files|filepattern to include in the package, code and assets)
  * engines (eg. `14.1`, `17.0`)
  * platforms (eg. `Mac-64`, `Windows-32`)
  * editions (eg. `Classic`, `Unicode`)
  * dependencies
  * [dependencies for development] (only relevant for Acre projects)
  * description
  * draft flag (?!)
  * [export]
  * [tags]
  * [project url]
  * [issue tracker url]
  * ...

### 1.2. Source code

The source code could be a single file or folder. The format of the source file(s) must be supported by `Link`.

## 2. Client

The client is mainly responsible for retrieving packages and linking them to the active workspace\|project. Both local registries and served registres are supported. The client has its own configuration file that keeps track of an ordered list of registries used. This simplifies operations by allowing the use of aliases to reference registries or implicit use of top listed registry.

In local registry mode, this would mean fetching a package using a specified URI (local filesystem or inter/intra net). With a registry server, this could be extended to allow for a package to be retrieved by name (`group/package`) and optional version number.

### 2.1. Pack (source_path target_path)

Takes a path to source and target folders. Inspects the `source_path` folder and validates that it contains the required files for a package (as defined above). If all ok, it then proceeds to add the required files (everything in source folder unless otherwise specified by the `files` property in the config file) to a zip archive. The zip file is named as defined above and placed in `target_path`.

### 2.2. Load (target_space uri)

This method simply loads a package and its dependencies into the active workspace and creates a reference to it in the target_space. Nothing persists between sessions as the state is not saved to file.

The package bundle is downloaded and unpacked into a local registry folder. The folder is keyed by the URI to allow for caching of packages. 

The code is then loaded into the active workspace into a namespace path defined by the group, name and version of the package as per:

`#.group.package.version`

The version is prefixed `v` and invalid APL characters are replaced by underscores (eg. `1.2.0-alpha` becomes `v1_2_0_alpha`).

A constant is created in the package space that specifies the path to the package folder on disk. This can be referenced by the package code when accessing package resources.

`#.group.package.version.HOME_DIR←'/path/to/package/'`

Finally, a reference to the direct dependency (the package added explicitly, not transitive dependecies) is created in the root.

`#.package←#.group.package.v1_2_0_alpha`

### 2.3. Load (uri project_space) - (only with Acre project)

As for `Load (uri)` but the package is added as a dependency in the project's config file to ensure it persists and is loaded next time the project is opened (handled by Acre?). Also, the reference to the package is created in the project space instead of the root.

### 2.4. Publish (source_path registry_uri)

First calls `Pack` to generate a package bundle and then posts it to the provided `registry_uri`. This could be either a serverless registry or registry server.

* If registry_uri is using the file scheme (starting with `file://`) then the client will do the work of copying the package to the correct path and add/update metadata for the package to index files.

* If registry_uri is using the http scheme (starting with `https://`) then the client will connect to server, authenticate and post the package.

## 3. Local registry

With a local registry all operations (eg. searching, listing and publishing) are executed by the client.

The folder structure of the registry is as follows:

```bash
registry/
  index/
  packages/
    GroupA/
      PackageA/
        GroupA-PackageA-v1.0.0.zip
        GroupA-PackageA-v1.0.1.zip
    GroupB/
      PackageB/
        GroupB-PackageB-v0.9.0.zip
        GroupB-PackageB-v0.9.1.zip
```

### 3.1. index

The index folder contains key metadata about the packages stored in the registry in a form optimised for handling typical operations, such as building dependency trees and searching packages. The inforamtion is extracted from the packages and the index can therefore be rebuilt if corrupted.

### 3.2. apl-project.json

The `apl-project.json` file is a structured file describing the package and its dependencies. 

```json
{
  "group": "GroupA",
  "name": "PackageA",
  "version": "1.0.0",
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
      "alias": "C5"
    }
  ]
}
```

## 4. Cache

When loading a package it is:

1. fetched (from registry) if not already in cache
1. unpacked into temp folder
1. content validated?
1. moved to cache
1. if installing in project, linked to project folder

```bash
/path/to/appdata/apm
  /cache
    /registry_alias
      /Group-Package-Version
```
Example: Cache with packages from registry with alias "public"

```bash
/usr/me/apm
  /cache
    /public
      /Carlisle-Rumba-4.0.0
      /Carlisle-Rumba-4.0.1
      /Carlisle-Rumba-4.1.0
```


Example: An Acre Project with dependency on Rumba package:

```bash
/usr/me/projectX
  /packages
    /Carlisle-Rumba-4.0.1 -> /usr/me/apm/cache/public/Carlisle-Rumba-4.0.1
```

Example: An Acre Project with dependency on Rumba package and FlipDB Acre Project:

```bash
/usr/me/projectX
  /packages
    /Carlisle-Rumba-4.0.1 -> /usr/me/apm/cache/public/Carlisle-Rumba-4.0.1
    /Carlisle-FlipDB-2.9.1-beta -> /usr/me/Carlisle/FlipDB
```

## Registry server

The main point of the server is to have a well defined API to manage packages (its main resource). On top of that it will need to manage users, organisations and the authentication/authorisation related to managing the packages.

### Users

* A user is identified by email.
* Reading: a user is only required when reading private packages.
* Publishing: a user is always required.
* A user can be assigned to one or more groups.

### [groups|organisations|units|scopes|teams]

We need to find a good name; for the moment in this document we use the term "group".

* A group has one or more users assigned to it.
* A group can have 0 or more packages.
* Only users in the group with write access are allowed to publish packages.
* A group name must be 
  1. a valid APL name
  1. consist of nothing but ASCII (**not** ANSI) characters

### API

* List all groups
* List all packages, possibly restricted by group and/or a pattern
* List all versions of all packages, possibly restricted by group and/or a pattern
* Get meta data for all packages, possibly restricted by group and/or a pattern
* Delete ?!
* Mark as inactive ?!

## Open questions

* Role of acre

  * Is the package manager acre agnostic?
  * Should acre be enhanced in order to become a package manager?

* How to save stuff on the server
  * Packages as such in the file system?
  * Meta data in JSON files?

* How to compress and uncompress?

  The client, when publishing, needs to compress according to the platform it is running on: zipping under Windows, tar otherwise.

  The server then needs to create the missing version and store both.

  The server also needs to save an uncompressed version of a package for faster searching operations, access to the meta data etc.

