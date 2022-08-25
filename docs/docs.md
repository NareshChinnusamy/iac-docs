# Canoja IaC Project

This project contains set of CloudFormation stacks used for creating deployment infrastructure in AWS Cloud using CDK(Cloud Development Kit).

## Getting started

### Requirements

In order to use this framework, you will need:

- AWS CLI v2
- AWS CDK v2.35.0

### Installation

#### AWS CLI v2

Setup AWS CLI using [this](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) link.

##### Configuring the AWS CLI

- Configure Breezeware AWS Account

  ```bash
  aws configure
    AWS Access Key ID [None]: <IAM_USER_ACCESS_ID> # IAM User's access id.
    AWS Secret Access Key [None]: <IAM_USER_ACCESS_KEY> # IAM User's access key.
    Default region name [None]: us-east-1 # Use 'us-east-1' as region.
    Default output format [None]: json # Use 'json' as output format.
  ```

- Configure Canoja AWS Account
  
  ```bash
  aws configure --profile canoja
    AWS Access Key ID [None]: <IAM_USER_ACCESS_ID> # IAM User's access id.
    AWS Secret Access Key [None]: <IAM_USER_ACCESS_KEY> # IAM User's access key.
    Default region name [None]: us-east-1 # Use 'us-east-1' as region.
    Default output format [None]: json # Use 'json' as output format.
  ```

For more details on configuring check [this](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html) link.

#### AWS CDK v2.35.0

Setup CDK in developer machine using [this](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html) link.

##### Useful commands for managing CDK stacks

- `mvn package`     compile and run tests
- `cdk ls`          list all stacks in the app
- `cdk synth`       emits the synthesized CloudFormation template
- `cdk deploy`      deploy this stack to your default AWS account/region
- `cdk diff`        compare deployed stack with current state
- `cdk docs`        open CDK documentation

## Project structure

- Directory `lambda` consists of python scripts for CI and CD Lambda Function.
- Directory `db`  consists of db dump sqls, Dockerfile and TaskDefinitionEnvironmentFile.env for all DB Ecs Servives in Canoja Project.
- Directory `fusion-auth` consists of TaskDefinitionEnvironmentFile.env for Fusion Auth Ecs Service.
- File `ecs-prod-key-pair.pem` is the SSH key pair for Ec2 Instances deployed using Ecs.
- File `cdk.json` file tells the CDK Toolkit how to execute your app.

- File `cdk.context.json` file handles the properties of the available stacks.

## Available CloudFormation stacks in the IaC project

This section lists the available cdk stacks in this IaC project and the deploy commands for each individual stack.

The ---profile canoja- in the deploy commands represent the customer aws account.

### Bootstrapping

  ```shell
  cdk bootstrap aws://<ACCOUNT-NUMBER>/<REGION>
  ```

- Deploying AWS CDK apps into an AWS environment requires that you provision resources the AWS CDK needs to perform the deployment.

- These resources include an Amazon S3 bucket for storing files and IAM roles that grant permissions needed to perform deployments. The process of provisioning these initial resources is called bootstrapping.

### Kms Key Setup

  ```shell
  cdk --profile canoja deploy KmsKeySetupStack
  ```

- Creates an custom kms key in the customer account for encrypting code pipeline artifact.

### Cross Account Iam Role

  ```shell
  cdk deploy canojaCrossAccountIamRoleSetupStack
  ```

- Creates an IAM role in development aws accout to provide access to code build artifacts(S3 bucket) to codepipeline in customer aws account.

### Open Telemetry For Xray Iam Role

  ```shell
  cdk deploy --profile canoja AWSDistroOpenTelemetryPolicyForXraySetupStack
  ```

- Creates an Iam role in customer aws account to provide access to xray and cloudwatch log services. It is used for ecs task role.

### CodeBuild S3 Bucket

  ```shell
  cdk deploy CanojaCodeBuildS3BucketSetupStack
  ```

- Creates an s3 bucket in development aws account for managing CI artifacts and cache.

### CodePipeline S3 Bucket

  ```shell
  cdk deploy --profile canoja CodePipelineS3BucketSetupStack
  ```

- Creates an s3 bucket in customer aws account for managing CD artifacts.

### CloudFormation S3 Bucket

  ```shell
  cdk deploy --profile canoja CanojaCloudFormationS3BucketSetupStack
  ```

- Creates an s3 bucket in customer aws account for managing CloudFormation template for CD process.

### Ecs S3 Bucket

  ```shell
  cdk deploy --profile canoja EcsS3BucketSetupStack
  ```

- Creates an s3 bucket in customer aws account for managing environment properties  for ecs.

### Event Bus

  ```shell
  cdk deploy --profile canoja EventBusSetupStack
  ```

- Creates an event bus in customer aws account for handling CD phase in the customer side.

### CodeBuild Template

  ```shell
  cdk synth CanojaCodeBuildSetupStack > code-build-setup-template.yml
  ```

- Creates an CodeBuild project cloudformation template and save the template in the following s3 bucket of development aws account: breezeware-cloudformation-artifacts-us-east-1/canoja/templates/ci/code-build-setup-template.yml-.

- Ci Lambda function will fetch the template and Creates the codebuild project.

Note: `Upload the CodeBuild CloudFormation template before deploying the CI lambda function.`

### CI Lambda Function

  ```shell
  cdk deploy CanojaCodeBuildLambdaFunctionSetupStack
  ```

- Creates an Lambda Function for managing CI flow in development aws account along with event bridge rule to trigger the lambda function.

### CodePipeline Template

  ```shell
  cdk synth CodePipelineSetupStack > code-pipeline-setup-template.yml
  ```

- Creates an CodePipeline cloudformation template and save the template in the following s3 bucket of customer aws account: -canoja-cloudformation-artifacts-us-east-1/templates/cd/code-pipeline-setup-template.yml-.

- Cd Lambda function will fetch the template and Creates the codebuild project.

Note: `Upload the CodePipeline CloudFormation template before deploying the CD lambda function.`

### CD Lambda Function

  ```shell
  cdk deploy --profile canoja CodePipelineLambdaFunctionSetupStack
  ```

- Creates an Lambda Function for managing CD flow in customer aws account along with event bridge rule to trigger the lambda function.
  
### Virtual Private Cloud

  ```shell
  cdk deploy --profile canoja CodePipelineLambdaFunctionSetupStack
  ```

- Create an VPC in customer aws account for managing ecs cluster and services in separate vpc.

### Cloud Map Service Discovery

  ```shell
  cdk deploy --profile canoja CloudMapServiceDiscoverySetupStack
  ```

- Creates an service discovery namespace for managing communications between different microservices inside a single vpc.

### Log Group

  ```shell
  cdk deploy --profile canoja OtelLogGroupSetupStack
  ```

- Creates an log group for collecting consolidated logs of OTEL side car collector.

### Acm Certificate

  ```shell
  cdk deploy --profile canoja ClusterAndAlbSecurityGroupSetupStack
  ```

- Creates an acm certificate in customer aws account for the particular domain.

  Note: `Deploy this stack only when there is no certificate available for the particular domain.`

### Security Groups

  ```shell
  cdk deploy --profile canoja ClusterAndAlbSecurityGroupSetupStack
  ```

- Creates security groups in customer aws account for ecs cluster and loadbalancer.

### Load Balancer

  ```shell
  cdk deploy --profile canoja LoadBalancerSetupStack
  ```

- Creates an Applcation LoadBalancer(ALB) along with the lsiteners for web, authority-web, bff services(authority-web-bff,web-bff), idm , application-svc and tagret groups for web, authority-web, authority-web-bff, web-bff, application-svc, and idm services.

### Ecs Cluster

  ```bash
  cdk deploy --profile canoja CanojaClusterProduction
  ```

- Creates an ecs cluster with two ASG capacity providers(Db, and App services).

### Ecr Repository

- Creates ecr repositories for managing private images of the all available services in customer project.
  - fusion-auth-db

    ```shell
    cdk deploy --profile canoja FusionAuthDbEcrRepository
    ```

  - canoja-svc-db

    ```shell
    cdk deploy --profile canoja CanojaSvcDbEcrRepository
    ```

  - canoja-user-svc-db

    ```shell
    cdk deploy --profile canoja CanojaUserSvcDbEcrRepository
    ```

  - canoja-application-svc-db

    ```shell
    cdk deploy --profile canoja CanojaApplicationSvcDbEcrRepository
    ```

  - fusion-auth-service

    ```shell
    cdk deploy --profile canoja FusionAuthServiceEcrRepository
    ```

  - canoja-svc-service

    ```shell
    cdk deploy --profile canoja CanojaSvcServiceEcrRepository
    ```

  - canoja-user-svc-service

    ```shell
    cdk deploy --profile canoja CanojaUserSvcServiceEcrRepository
    ```

  - canoja-application-svc-service

    ```shell
    cdk deploy --profile canoja CanojaApplicationSvcServiceEcrRepository
    ```

  - canoja-web-bff-service

    ```shell
    cdk deploy --profile canoja CanojaWebBffServiceEcrRepository
    ```

  - canoja-authority-web-bff-service

    ```shell
    cdk deploy --profile canoja CanojaAuthorityWebBffServiceEcrRepository
    ```

  - canoja-auth-svc-service

    ```shell
    cdk deploy --profile canoja CanojaAuthSvcServiceEcrRepository
    ```

  - canoja-web-service

    ```shell
    cdk deploy --profile canoja CanojaWebServiceEcrRepository
    ```

  - canoja-authority-web-service

    ```shell
    cdk deploy --profile canoja CanojaAuthorityWebServiceEcrRepository
    ```

### Ecs Services

- Creates ecs services for the available services in canoja project.
- Properties for the particular services managed using cdk context while deploying stacks.
  - fusion-auth-db-service

    ```shell
    cdk --profile canoja --context default/ecs/service:name=fusion-auth-db --context default/ecs/service:loggroup:name=/ecs/canoja/fusion-auth-db --context default/ecs/service:taskdefinition:name=fusion-auth-db-taskdefinition  --context default/ecs/service:applicationcontainer:name=fusion-auth-db --context default/ecs/service:applicationimage:name=fusion-auth-db --context default/ecs/service:applicationimage:tag=latest  --context default/ecs/service:cloudmapservice:name=fusion-auth-db-service  --context default/ecs/service:volume:name=fusion-auth-db  --context default/ecs/service:volume:container:path=/var/lib/postgresql/data deploy FusionAuthDbService
    ```

  - canoja-svc-db-service

    ```shell
    cdk --profile canoja --context default/ecs/service:name=canoja-svc-db --context default/ecs/service:loggroup:name=/ecs/canoja/canoja-svc-db --context default/ecs/service:taskdefinition:name=canoja-svc-db-taskdefinition  --context default/ecs/service:applicationcontainer:name=canoja-svc-db --context default/ecs/service:applicationimage:name=canoja-svc-db --context default/ecs/service:applicationimage:tag=latest  --context default/ecs/service:cloudmapservice:name=canoja-svc-db-service  --context default/ecs/service:volume:name=canoja-svc-db  --context default/ecs/service:volume:container:path=/var/lib/postgresql/data deploy CanojaSvcDbService
    ```

  - canoja-user-svc-db-service

    ```shell
    cdk --profile canoja --context default/ecs/service:name=canoja-user-svc-db --context default/ecs/service:loggroup:name=/ecs/canoja/canoja-user-svc-db --context default/ecs/service:taskdefinition:name=canoja-user-svc-db-taskdefinition  --context default/ecs/service:applicationcontainer:name=canoja-user-svc-db --context default/ecs/service:applicationimage:name=canoja-user-svc-db --context default/ecs/service:applicationimage:tag=latest  --context default/ecs/service:cloudmapservice:name=canoja-user-svc-db-service  --context default/ecs/service:volume:name=canoja-user-svc-db  --context default/ecs/service:volume:container:path=/var/lib/postgresql/data deploy CanojaUserSvcDbService
    ```

  - canoja-application-svc-db-service

    ```shell
    cdk --profile canoja --context default/ecs/service:name=canoja-application-svc-db --context default/ecs/service:loggroup:name=/ecs/canoja/canoja-application-svc-db --context default/ecs/service:taskdefinition:name=canoja-application-svc-db-taskdefinition  --context default/ecs/service:applicationcontainer:name=canoja-application-svc-db --context default/ecs/service:applicationimage:name=canoja-application-svc-db --context default/ecs/service:applicationimage:tag=latest  --context default/ecs/service:cloudmapservice:name=canoja-application-svc-db-service  --context default/ecs/service:volume:name=canoja-application-svc-db  --context default/ecs/service:volume:container:path=/var/lib/postgresql/data deploy CanojaApplicationSvcDbService
    ```

  - fusion-auth-service

    ```shell
    cdk --profile canoja --context default/ecs/service:name=fusion-auth --context default/ecs/service:loggroup:name=/ecs/canoja/fusion-auth --context default/ecs/service:taskdefinition:name=fusion-auth-taskdefinition  --context default/ecs/service:applicationcontainer:name=fusion-auth --context default/ecs/service:applicationimage:name=fusion-auth --context default/ecs/service:applicationimage:tag=1.31.0  --context default/ecs/service:cloudmapservice:name=fusion-auth-service deploy FusionAuthService
    ```
    
  - canoja-svc-service

    ```shell
    cdk --profile canoja --context default/ecs/service:name=canoja-svc --context default/ecs/service:loggroup:name=/ecs/canoja/canoja-svc --context default/ecs/service:taskdefinition:name=canoja-svc-taskdefinition  --context default/ecs/service:applicationcontainer:name=canoja-svc --context default/ecs/service:applicationimage:name=canoja-svc --context default/ecs/service:applicationimage:tag=<service-latest-version>  --context default/ecs/service:cloudmapservice:name=canoja-svc-service deploy CanojaSvcService
    ```

  - canoja-user-svc-service

    ```shell
    cdk --profile canoja --context default/ecs/service:name=canoja-user-svc --context default/ecs/service:loggroup:name=/ecs/canoja/canoja-user-svc --context default/ecs/service:taskdefinition:name=canoja-user-svc-taskdefinition  --context default/ecs/service:applicationcontainer:name=canoja-user-svc --context default/ecs/service:applicationimage:name=canoja-user-svc --context default/ecs/service:applicationimage:tag=<service-latest-version>  --context default/ecs/service:cloudmapservice:name=canoja-user-svc-service deploy CanojaUserService
    ```

  - canoja-application-svc-service

    ```shell
    cdk --profile canoja --context default/ecs/service:name=canoja-application-svc --context default/ecs/service:loggroup:name=/ecs/canoja/canoja-application-svc --context default/ecs/service:taskdefinition:name=canoja-application-svc-taskdefinition  --context default/ecs/service:applicationcontainer:name=canoja-application-svc --context default/ecs/service:applicationimage:name=canoja-application-svc --context default/ecs/service:applicationimage:tag=<service-latest-version>  --context default/ecs/service:cloudmapservice:name=canoja-application-svc-service deploy CanojaApplicationService
    ```

  - canoja-web-bff-service

    ```shell
    cdk --profile canoja --context default/ecs/service:name=canoja-web-bff --context default/ecs/service:loggroup:name=/ecs/canoja/canoja-web-bff --context default/ecs/service:taskdefinition:name=canoja-web-bff-taskdefinition  --context default/ecs/service:applicationcontainer:name=canoja-web-bff --context default/ecs/service:applicationimage:name=canoja-web-bff --context default/ecs/service:applicationimage:tag=<service-latest-version>  --context default/ecs/service:cloudmapservice:name=canoja-web-bff-service deploy CanojaWebBffService
    ```

  - canoja-authority-web-bff-service

    ```shell
    cdk --profile canoja --context default/ecs/service:name=canoja-authority-web-bff --context default/ecs/service:loggroup:name=/ecs/canoja/canoja-authority-web-bff --context default/ecs/service:taskdefinition:name=canoja-authority-web-bff-taskdefinition  --context default/ecs/service:applicationcontainer:name=canoja-authority-web-bff --context default/ecs/service:applicationimage:name=canoja-authority-web-bff --context default/ecs/service:applicationimage:tag=<service-latest-version>  --context default/ecs/service:cloudmapservice:name=canoja-authority-web-bff-service deploy CanojaAuthorityWebBffService
    ```

  - canoja-auth-svc-service

    ```shell
    cdk --profile canoja --context default/ecs/service:name=canoja-auth-svc --context default/ecs/service:loggroup:name=/ecs/canoja/canoja-auth-svc --context default/ecs/service:taskdefinition:name=canoja-auth-svc-taskdefinition  --context default/ecs/service:applicationcontainer:name=canoja-auth-svc --context default/ecs/service:applicationimage:name=canoja-auth-svc --context default/ecs/service:applicationimage:tag=<service-latest-version>  --context default/ecs/service:cloudmapservice:name=canoja-auth-svc-service deploy CanojaAuthService
    ```

  - canoja-web-service

    ```shell
    cdk --profile canoja --context default/ecs/service:name=canoja-web --context default/ecs/service:loggroup:name=/ecs/canoja/canoja-web --context default/ecs/service:taskdefinition:name=canoja-web-taskdefinition  --context default/ecs/service:applicationcontainer:name=canoja-web --context default/ecs/service:applicationimage:name=canoja-web --context default/ecs/service:applicationimage:tag=<service-latest-version> deploy CanojaWebService
    ```

  - canoja-authority-web-service

    ```shell
    cdk --profile canoja --context default/ecs/service:name=canoja-authority-web --context default/ecs/service:loggroup:name=/ecs/canoja/canoja-authority-web --context default/ecs/service:taskdefinition:name=canoja-authority-web-taskdefinition  --context default/ecs/service:applicationcontainer:name=canoja-authority-web --context default/ecs/service:applicationimage:name=canoja-authority-web --context default/ecs/service:applicationimage:tag=<service-latest-version> deploy CanojaAuthorityWebService
    ```
