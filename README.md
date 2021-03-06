# ServiceNow API Jenkins Plugin

This is a Jenkins plugin that allows a workflow step
to automatically build API requests for the ServiceNow API and return appropriately parsed
values based on the type of API called.

It defines a set of configuration values that can be used to identify the ServiceNow instance
information and configure the API requests.

# Plugin Usage

## `serviceNow_createChange`

### Required Parameters

`serviceNowConfiguration`
* `instance` - Instance of service-now to connect to (https://<instance>.service-now.com)
* `producerId` - Producer ID of the standard change to create

`credentialsId` - Jenkins credentials for Username with Password credentials (or Vault Role Credentials if including vaultConfiguration below)


### Optional Parameters

`vaultConfiguration`
* `url` - Vault url
* `path` - Vault path to get authentication values

### Response

`result`
* `sys_id`
* `number`
* `record`
* `redirect_portal_url`
* `redirect_url`
* `table`
* `redirect_to`

### Example
Create a service-now change and capture the sys_id and change number
```groovy
def response = serviceNow_createChange serviceNowConfiguration: [instance: 'exampledev', producerId: 'ls98y3khifs8oih3kjshihksjd'], credentialsId: 'jenkins-vault', vaultConfiguration: [url: 'https://vault.example.com:8200', path: 'secret/for/service_now/']
def jsonSlurper = new JsonSlurper()
def createResponse = jsonSlurper.parseText(response.content)
def sysId = createResponse.result.sys_id
def changeNumber = createResponse.result.number
```

## `serviceNow_UpdateChange`

### Required Parameters

`serviceNowConfiguration`
* `instance` - Instance of service-now to connect to (https://<instance>.service-now.com)

`credentialsId` - Jenkins credentials for Username with Password credentials (or Vault Role Credentials if including vaultConfiguration below)

`serviceNowItem`
* `table` - Table of the item to be updated (ex. change_request, change_task)
* `sysId` - SysId of the item to be updated
* `body` - Json message (as String) of the properties and values to be updated

### Optional Parameters

`vaultConfiguration`
* `url` - Vault url
* `path` - Vault path to get authentication values

### Response

`result`

### Example
Update a service-now change with a short description and description
```groovy
def messageJson = new JSONObject()
messageJson.putAll([
                short_description: 'My change',
                description: 'My longer description of the change'
        ])
def response = serviceNow_UpdateChangeItem serviceNowConfiguration: [instance: 'exampledev'], credentialsId: 'jenkins-vault', serviceNowItem: [table: 'change_request', sysId: 'adg98y29thukwfd97ihu23', body: messageJson.toString()], vaultConfiguration: [url: 'https://vault.example.com:8200', path: 'secret/for/service_now/']

```

## `serviceNow_getChangeState`

### Required Parameters

`serviceNowConfiguration`
* `instance` - Instance of service-now to connect to (https://<instance>.service-now.com)

`credentialsId` - Jenkins credentials for Username with Password credentials (or Vault Role Credentials if including vaultConfiguration below)

`serviceNowItem`
* `sysId` - SysId of the change request

### Optional Parameters

`vaultConfiguration`
* `url` - Vault url
* `path` - Vault path to get authentication values

### Response

`state` - String representation of the state (@see [ServiceNowStates](src/main/java/org/jenkinsci/plugins/servicenow/util/ServiceNowStates.java))

### Example
Get the current state of a service-now change
```groovy
def response = serviceNow_UpdateChangeItem serviceNowConfiguration: [instance: 'exampledev'], credentialsId: 'jenkins-vault', serviceNowItem: [sysId: 'adg98y29thukwfd97ihu23'], vaultConfiguration: [url: 'https://vault.example.com:8200', path: 'secret/for/service_now/']
echo response //NEW
```

## `serviceNow_getCTask`

### Required Parameters

`serviceNowConfiguration`
* `instance` - Instance of service-now to connect to (https://<instance>.service-now.com)

`credentialsId` - Jenkins credentials for Username with Password credentials (or Vault Role Credentials if including vaultConfiguration below)

`serviceNowItem`
* `sysId` - SysId change to get CTask from
* `ctask` - String representation of CTask (see [ServiceNowCTasks](src/main/java/org/jenkinsci/plugins/servicenow/util/ServiceNowCTasks.java))

### Optional Parameters

`vaultConfiguration`
* `url` - Vault url
* `path` - Vault path to get authentication values

### Response

`result` - an array of results
* `sys_id`
* `number`
* `short_description`

### Example
Get ctask of a service-now change
```groovy
def response = serviceNow_getCTask serviceNowConfiguration: [instance: 'exampledev'], credentialsId: 'jenkins-vault', serviceNowItem: [sysId: 'agsdh0wehosid9723h30h', ctask: 'UAT_TESTING'], vaultConfiguration: [url: 'https://vault.example.com:8200', path: 'secret/for/service_now/']
def ctaskSysId = createResponse.result[0].sys_id
def ctaskNumber = createResponse.result[0].number
```

## `serviceNow_attachFile`

### Required Parameters

`serviceNowConfiguration`
* `instance` - Instance of service-now to connect to (https://<instance>.service-now.com)

`credentialsId` - Jenkins credentials for Username with Password credentials (or Vault Role Credentials if including vaultConfiguration below)

`serviceNowItem`
* `sysId` - SysId to attach to (can be Change Request or CTask)
* `body` - String body of file to attach

### Optional Parameters

`vaultConfiguration`
* `url` - Vault url
* `path` - Vault path to get authentication values

### Response

`result`

### Example
Attach file to item in service-now
```groovy
def myFile = readFile file: 'my-test-file.txt'
serviceNow_attachFile serviceNowConfiguration: [instance: 'exampledev'], credentialsId: 'jenkins-vault', serviceNowItem: [sysId: 'agsdh0wehosid9723h30h', body: myFile], vaultConfiguration: [url: 'https://vault.example.com:8200', path: 'secret/for/service_now/']
```

## `serviceNow_attachZip`

Attach a zip file (from the current directory) to a service-now item

### Required Parameters

`serviceNowConfiguration`
* `instance` - Instance of service-now to connect to (https://<instance>.service-now.com)

`credentialsId` - Jenkins credentials for Username with Password credentials (or Vault Role Credentials if including vaultConfiguration below)

`serviceNowItem`
* `sysId` - SysId to attach to (can be Change Request or CTask)
* `filename` - Filename of the zip file to attach

### Optional Parameters

`vaultConfiguration`
* `url` - Vault url
* `path` - Vault path to get authentication values

### Response

`result`

### Example
Attach zip to item in service-now
```groovy
zip zipFile: 'my-zip-file.zip', glob: '*.txt'
serviceNow_attachZip serviceNowConfiguration: [instance: 'exampledev'], credentialsId: 'jenkins-vault', serviceNowItem: [sysId: 'agsdh0wehosid9723h30h', filename: 'my-zip-file.zip'], vaultConfiguration: [url: 'https://vault.example.com:8200', path: 'secret/for/service_now/']
```