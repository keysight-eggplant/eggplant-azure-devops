# Eggplant Runner in CI/CD with Azure DevOps

Eggplant Runner currently provides "Run Test Config" as its main action.

## Using run-test-config.yml in your pipeline

In order to use the Eggplant Runner with Azure DevOps, you need to add either one of these to your Azure Pipelines .yml:

1. Referencing our `run-test-config` directly from your pipeline .yml:

```yaml
trigger:
  - main
  
resources:
  repositories:
    - repository: sourceRepo
      type: github
      name: TestPlant/eggplant-azure-devops
      endpoint: # your service connection name. Details below under Notes(4).
jobs:
  - job: Run_Test_Config
    strategy:
      maxParallel: 1
      matrix:
        linux:
          imageName: 'ubuntu-latest'
        mac:
          imageName: 'macOS-latest'
        windows:
          imageName: 'windows-latest'
    pool: 
      vmImage: $(imageName)
    steps:
      - template: templates/run-test-config.yml@sourceRepo
        parameters:
          serverURL: # Required. Details below
          testConfigID: # Required. Details below
          clientSecret: # Required. Details below
```

**OR**

2. Cloning `run-test-config.yml` in your own repository and referencing to it.<br />
This would also allow you to modify your own copy of the action script to your liking/needs:

```yaml
trigger:
  - main
  
jobs:
  - job: Run_Test_Config
    strategy:
      maxParallel: 1
      matrix:
        linux:
          imageName: 'ubuntu-latest'
        mac:
          imageName: 'macOS-latest'
        windows:
          imageName: 'windows-latest'
    pool: 
      vmImage: $(imageName)
    steps:
      - template: templates/run-test-config.yml
        parameters:
          serverURL: # Required. Details below
          testConfigID: # Required. Details below
          clientSecret: # Required. Details below
```

## Inputs

### `serverURL`
**Required** The URL of the DAI server, e.g. `http://localhost:8000`.

### `testConfigID`
**Required** The ID of the test config that you want to run, e.g. `09c48b7d-fc5b-481d-af80-fcffad5d9587`.

### `clientSecret`
**Required** The client secret to use to authenticate with the DAI server, e.g. `e9c15662-8c1b-472e-930d-aa0b11726093`.<br />
             Alternatively, you could set a pipeline secret and refer to it like below:<br />
             `clientSecret: $(DAI_CLIENT_SECRET)`.<br />
             Reading: https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch#secret-variables
             
### `clientID`
**Optional** The client ID to use to authenticate with the DAI server.<br />
**Default:** `client:dai:agent:integration`

### `requestTimeout`
**Optional** The timeout in seconds for each HTTP request to the DAI server<br />
**Default:** `30`

### `requestRetries`
**Optional** The number of times to attempt each HTTP request to the DAI server<br />
**Default:** `5`

### `backoffFactor`
**Optional** The exponential backoff factor between each HTTP request<br />
**Default:** `0.5`

### `pollInterval`
**Optional** The number of seconds to wait between each call to the DAI server<br />
**Default:** `5`

### `logLevel`
**Optional** The logging level<br />
**Default:** `INFO`

### `CACertPath`
**Optional** The path to an alternative Certificate Authority pem file<br />

### `dryRun`
**Optional** Dry Run mode only validates the parameters without executing a test config run. It does not require a connection to the DAI server.<br />
**Default:** `False`.


## Notes

1. This pipeline .yml file needs to in the root directory of your repository.<br />


2. On `strategy: max-parallel: 1`: SUT(System Under Test) is locked for one test config run at a time.<br />
Hence, we can only do unilateral testing.


3. Eggplant Runner supports these OS: Linux, Windows, MacOS.


4. In order to reference our template `run-test-config.yml` script directly, you need to create a GitHub Service Connection.<br />
In Azure DevOps page: `Project settings > Pipelines > Service connections > New service connection > GitHub`.<br />
Then, populate `resources: repositories: endpoint` with the newly created `Service connection name`.
