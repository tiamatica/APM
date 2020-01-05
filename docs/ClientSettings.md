# Client Settings

There is a client configuration file ([apm-client.json](./apm-client-schema.json)) that keeps track of global user settings. Below is an example apm-client.json file.

## Properties

| property | type | desc |
| --- | --- | --- |
| `registries` | array | Array of registry objects. This allows the client to reference registries with aliases, without providing the full uri. |


## Example

```json
{
    "registries": [
        {
            "alias": "local",
            "uri": "file://C:/Users/me/myregistry",
        },
        {
            "alias": "dyalog",
            "uri": "https://packages.dyalog.com",
        },
        {
            "alias": "myCompany",
            "uri": "https://packages.my-company.com",
            "user": "me@my-company.com"
        }
    ]
}
```
