# Dr. Max Gitlab AzDO Migration

Migrates gitlab repositories with open merge requests from Gitlab to Azure DevOps

## Development

- `$ make dep` initiates go modules
- `$ make test` checks the code

### Important Dependencies

- https://github.com/xanzy/go-gitlab used for Gitlab communication
- https://github.com/microsoft/azure-devops-go-api used for AzDO communication
- https://gopkg.in/alecthomas/kingpin.v2 for cli input processing

## Run!

- `$ make` prepares win/linux/mac binaries into bin folder
- Use your preffered binary with following arguments

### Run Options


| `--custom-gitlab-api-url` | string (**optional**) | If specified, this will connect to a self-hosted GitLab instance instead of Gitlab.com. Format should be "https://gitlab.myorg.com/api/v4"                             |

### Service endpoint configuration

If you're importing private repositories you need to configure [Service Endpoint](https://docs.microsoft.com/en-us/azure/devops/extend/develop/service-endpoints?view=azure-devops) in AzDO project to authenticate.

1. In your Azure DevOps project navigate to settings
2. In the left menu click *Pipelines* > *Service connections*
3. Click *New service connection*
4. Select *Generic*
5. Fill following configuration
   1. Server URL `https://gitlab.com`
   2. Username `gitlab.com`
   3. Password/Token Key `YOUR GITLAB API TOKEN`
   4. Service connection name `AzDO migration` (or any other descriptive text)
6. Click Save
7. Click on your service endpoint and your identificator will be visible in the URL
   `https://dev.azure.com/MYORG/MYPROJECT/_settings/adminservices?resourceId=**SERVICE_ENDPOINT**`
8. You can remove the service endpoint once you're done importing your repositories.

### Config File

The structure of config file is as follows:

```
{
  "projects": [
    {
      "gitlabID": 622148,
      "azdoProject": "my-project",
      "migrateMRs": true
    },
    #...
  ]
}
```

For each project you must (i.e. they're required) to specify three attributes:

- **gitlabID** - (_int_) ID of your gitlab project
- **azdoProject** - (_string_) name of the project where repository should be migrated to
- **migrateMRs** - (_bool_) whether or not active Merge requests should be migrated as well

## Known issues

- **Empty repositories** - repositories with no branches are not transferred due to limitation on Azure DevOps import request procedure
- **Author of Pull request/discussion** - It's nearly impossible to transfer author between the services thus:
  - All pull requests are created under the user to which azure access token belongs to - we suggest creating specific user for the migration
  - All pull request discussions are also authored to the access token user
  - However for every item (both pull requests and discussions/comments) first line contains info on the original author as well as reference to their gitlab account 
- **Azure DevOps import notifications** - for every import request azure will send you notification of successful import. If you're migrating huge amount of repositories, brace yourselves/your inboxes
- **Existing disabled repository** - it's not possible to fetch/remove existing disabled repository via Azure DevOps api.
