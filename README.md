<img src="https://www.eggplantsoftware.com/hubfs/Branding/Keysight-Eggplant-Logo_RGB_full-color.svg" width="300px"/>

# Eggplant DAI Runner with Azure DevOps

## Introduction

The Eggplant DAI Runner is an [Eggplant DAI](https://www.eggplantsoftware.com/digital-automation-intelligence) integration tool that enables the functionality to launch DAI tests from within a Azure DevOps pipeline. You can use it to continuously test your application's [model-based approach to testing](https://docs.eggplantsoftware.com/docs/dai-using-eggplant-dai/).  For more information about Eggplant, visit https://www.eggplantsoftware.com.

The core integration of the **Eggplant DAI Runner** are with [**DAI Test Configuration**](https://docs.eggplantsoftware.com/docs/dai-test-configuration/). **Eggplant DAI Runner** basically will communicate with the API services provided by **Eggplant DAI** to perform test configuration execution.
Eggplant Runner currently provides "Run Test Config" as its main action.

## Using run-test-config.yml in your pipeline

In order to use the Eggplant Runner with Azure DevOps, you need to add this to your Azure Pipelines .yml file:<br />
Reading: https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops#use-other-repositories

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

## Inputs

### `serverURL`
**Required** The URL of the DAI server, e.g. `http://localhost:8000`.

### `testConfigID`
**Required** The ID of the test config that you want to run, e.g. `09c48b7d-fc5b-481d-af80-fcffad5d9587`.

Test configuration ID can be obtain by go to test config > look for a particular test config > test config id can be obtain from url.
![image](https://user-images.githubusercontent.com/101400930/165948106-3bcac6b6-194a-468c-84ab-b1ea619d90de.png)

### `clientSecret`
**Required** The client secret to use to authenticate with the DAI server, e.g. `e9c15662-8c1b-472e-930d-aa0b11726093`.<br />
             Alternatively, you could set a pipeline secret and refer to it like below:<br />
             `clientSecret: $(DAI_CLIENT_SECRET)`.<br />
             Reading: https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch#secret-variables

The **DAI Client Secret** can be obtain by go to http://kc-localhost:8000/auth > clients > search for client:dai:agent:integration > credential > secret
![image](https://user-images.githubusercontent.com/101400930/167881013-7b164d9e-41f1-4ce2-b08a-21704acb9d36.png)
             
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


4. In order to reference our template `run-test-config.yml` script, you need to create a GitHub Service Connection.<br />
In Azure DevOps page: `Project settings > Pipelines > Service connections > New service connection > GitHub`.<br />
Then, populate `resources: repositories: endpoint` with the newly created `Service connection name`.
