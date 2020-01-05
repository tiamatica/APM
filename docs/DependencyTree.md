# Dependency tree

When creating a project with dependencies on packages from package registries, the project needs to specify the fully qualified names (Group-Name-Version) of the dependencies in a config file (`apl-project.json`). A companion file (`apl-dep-tree.json`) is generated that defines the resolved package paths on disk as well as aliases requested.

## Example

A package with the following dependency tree is installed in a project:

* Carlisle-Rumba-v2.1.0
  * APLTeam-Utils-v2.0.0
  * Dyalog-Conga-v3.1.3
    * APLTeam-Utils-v1.9.10

The project defines the dependency in its `apl-project.json` file and specifies the alias `R` (`@R` suffix):

```json
{
    "dependencies": [ "Carlisle-Rumba-v2.1.0@R" ] 
}
```

APM resolves the dependencies recursively and logs them to file (`apl-dep-tree.json`):

```json
[
    {
        "id": "Carlilse-Rumba-v2.1.0",
        "dependencies": [
            "APLTeam-Utils-v2.0.0"
            "Dyalog-Conga-v3.1.3",
        ] 
    },
    {
        "id": "APLTeam-Utils-v2.0.0",
        "dependencies": [] 
    },
    {
        "id": "Dyalog-Conga-v3.1.3",
        "dependencies": [ "APLTeam-Utils-v1.9.10" ] 
    },
    {
        "id": "APLTeam-Utils-v1.9.10",
        "dependencies": [] 
    },
]
```

The packages are fetched into cache and then sym linked into project folder:

```bash
/project
  /packages
    /APLteam-Utils-v1.9.10 -> {APM_CACHE}/registry/AplTeam-Utils/v1.9.10
    /APLteam-Utils-v2.0.0  -> {APM_CACHE}/registry/AplTeam-Utils/v2.0.0
    /Carlilse-Rumba-v2.1.0 -> {APM_CACHE}/registry/Carlisle-Rumba/v2.1.0
    /Dyalog-Conga-v3.1.3   -> {APM_CACHE}/registry/Dyalog-Conga/v3.1.3
```

When opening the project, both files are interrogated to load and establish the code in the workspace. If the `apl-deps-tree.json` file is missing, it is generated. If it is present, APM could prompt for an updates check.

