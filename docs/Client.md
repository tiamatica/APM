# Client

The Client module implements most of the features required in the framework and is the main module in APM. It has its own configuration file ([ClientSettings](./ClientSettings.md)) to keep track of user preferences.

## 1. API

### 1.1. (rc msg zipfile) ← PackProject (project_path target_path)

Takes a path to a project folder and a target folder. Inspects the project folder and validates that it contains the required files for a package. If all ok, it then proceeds to add the required files to a zip archive. 

By default, everything in project folder is added to the archive unless the `packageFiles` property is defined in the project config file. This property is a whitelist of files and folders that should be included in a package.

The zip file is named `{group}-{name}-{version}.zip` and placed in `target_path`.

| parameter | type | desc |
| --- | --- | --- |
| `project_path` | string | Path to project folder. |
| `target_path` | string | Path to target folder where zipped package is to be created. |

| output | type | desc |
| --- | --- | --- |
| `rc` | number | Return code (0 = success, otherwise failed). |
| `msg` | string | Error message (empty on success). |
| `zipfile` | string | Full path to zipfile created. |


### 1.2. (rc msg) ← {registry} PublishPackage (package)

Publish `package` to `registry`.

The client uses the [Registry.AddPackage](./Registry.md###-1.2.-(rc-msg)-←-AddPackage-(packageZip)) method to publish the package to the registry.

| parameter | type | desc |
| --- | --- | --- |
| `package` | string\|object | If string then it indicates the path to a project folder or a package zip file. If object, it indicates a project space. |
| `registry` | string | (Optional) URI or alias for a registry (default to `defaultRegistry` in client settings). |

| output | type | desc |
| --- | --- | --- |
| `rc` | number | Return code (0 = success, otherwise failed). |
| `msg` | string | Error message (empty on success). |


### 1.3. (rc msg pkgref) ← {registry} LoadPackage (package target_space)

This method loads a `package` and its dependencies into the active workspace and creates a reference to it in the target space. Nothing persists between sessions as the state is not saved to file.

1. If `package` is using the file scheme (starting with `file://`) then it is pointing to a project folder or package zip file. 

    1. For a folder, the client will load the project directly from the folder
    1. For a zip file, the client will unpack into temp folder and load it from there.

1. If `package` is using the HTTP scheme (starting with `http(s)://`) then it is pointing to a package zip file. The client will download, unpack into temp folder and load the package. 

1. If `package` is using neither of the HTTP or file schemes, then it is a [Package ID](./Project.md##-1.2.-Project-ID) for a package in a registry. The client will use the [Registry.GetPackage](./Registry.md###-1.5.-(rc-msg-zipFile)-←-GetPackage-(packageID)) method to retrieve the package, unpack into temp folder and load the package. 

1. If `registry` is using the file scheme (starting with `file://`) then the client will use the [Registry.AddPackage](./Registry.md###-1.2.-(rc-msg)-←-AddPackage-(packageZip)) method directly.

* If `registry` is using the http scheme (starting with `http(s)://`) then the client will use the [Server.Add package](./Server.md###-3.2.-Add-package) API.


The package bundle is retrieved from `registry` and unpacked into a cache folder. The folder is keyed by the `package` URI to allow for caching of packages. 

The code is then loaded into the active workspace into a namespace path defined by the group, name and version of the package as per:

`#.group.name.version`

The version is prefixed `v` and invalid APL characters are replaced by underscores (eg. `1.2.0-alpha` becomes `v1_2_0_alpha`).

A constant is created in the package space that specifies the path to the package folder on disk. This can be referenced by the package code when accessing package resources.

`#.group.name.version.HOME_DIR←'/path/to/package/'`

Finally, a reference to the direct dependency (the package added explicitly, not transitive dependecies) is created in the target space.

`#.package←#.group.name.v1_2_0_alpha`

