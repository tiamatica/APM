# Registry

The Registry module manages the storage and retrieval of packages. No authentication is performed in this module.

When creating an instance, the URI provided dictates if operations are executed locally (if file scheme used) or proxied via a server (if HTTP scheme used).

## 1. API

### 1.1. (rc msg reg) ← New (options)

Create a new Registry instance given the options in the argument. Returns reference to registry instance.

`options` is a namespace with parameters.

| parameter | type | desc |
| --- | --- | --- |
| `type` | string | Type of registry to create. Only `local` supported. |
| `uri` | string | URI of registry. Uses file or HTTP schema. |

| output | type | desc |
| --- | --- | --- |
| `rc` | number | Return code (0 = success, otherwise failed). |
| `msg` | string | Error message (empty on success). |
| `reg` | object | Instance of a Registry. |

### 1.2. (rc msg) ← CreateIndex (force)

Create index for registry.

| parameter | type | desc |
| --- | --- | --- |
| `force` | number [0,1] | If `1` forces re-creation of index even if already exists. |

| output | type | desc |
| --- | --- | --- |
| `rc` | number | Return code (0 = success, otherwise failed). |
| `msg` | string | Error message (empty on success). |


### 1.3. (rc msg packages) ← ListPackages (options)

Search registry for packages matching criteria set out in options. The result is an array of packages that match all the criteria (AND).

`options` is a namespace with parameters.

| parameter | type | desc |
| --- | --- | --- |
| `group` | string | RegEx for package group to match. |
| `name` | string | RegEx for package name to match. |
| `version` | string | RegEx for package version to match. |

| output | type | desc |
| --- | --- | --- |
| `rc` | number | Return code (0 = success, otherwise failed). |
| `msg` | string | Error message (empty on success). |
| `packages` | array | Array of [package config objects](./apl-project-schema.json) |


### 1.4. (rc msg) ← AddPackage (packageZip)

Add a package to the registry.

In proxy mode the client will use the [Server.Add package](./Server.md###-3.2.-Add-package) API.

| parameter | type | desc |
| --- | --- | --- |
| `packageZip` | string | Path to zip file containing package. |

| output | type | desc |
| --- | --- | --- |
| `rc` | number | Return code (0 = success, otherwise failed). |
| `msg` | string | Error message (empty on success). |


### 1.5. (rc msg zipFile) ← GetPackage (packageID)

Retrieve a package from the registry. Returns the path to a package zip file.

In proxy mode the client will use the [Server.Get package](./Server.md###-3.3.-Get-package) API.

| parameter | type | desc |
| --- | --- | --- |
| `packageID` | string |  A [Package ID](./Project.md##-1.2.-Project-ID). |

| output | type | desc |
| --- | --- | --- |
| `rc` | number | Return code (0 = success, otherwise failed). |
| `msg` | string | Error message (empty on success). |
| `zipFile` | string | File path to package zipfile. |


### 1.6. (rc msg tree) ← DependencyTree (packages)

Resolve the dependency tree for the array of packages provided. The tree returned is using the `exact` version strategy.

| parameter | type | desc |
| --- | --- | --- |
| `packages` | array | Array of [package config objects](./apl-project-schema.json) |

| output | type | desc |
| --- | --- | --- |
| `rc` | number | Return code (0 = success, otherwise failed). |
| `msg` | string | Error message (empty on success). |
| `tree` | array | Nested array of [package config objects](./apl-project-schema.json). |

