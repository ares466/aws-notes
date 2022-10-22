# Contents

- [CI/CD](#cicd)
- [CodePipeline](#codepipeline)
  - [Troubleshooting](#codepipeline-troubleshooting)
- [CodeCommit](#codecommit)
- [CodeBuild](#codebuild)
  - [CodeBuild Phases](#codebuild-phases)
- [CodeDeploy](#codedeploy)
  - [CodeDeploy Lifecycle Hooks](#codedeploy-lifecycle-hooks)
  - [CodeDeploy Across Accounts](#codedeploy-across-accounts)
- [ECR](#ecr)

# CI/CD

Conceptually, there are four stages for code:
- Code (e.g., CodeCommit)
- Build (e.g., CodeBuild)
- Test (e.g., CodeBuild)
- Deploy (e.g., CodeDeploy)

AWS CodePipeline is the orchestrator to facilitate the code pipeline.

# CodePipeline

**AWS CodePipeline** is a continuous delivery service you can use to model, visualize, and automate the steps required to release your software. You can quickly model and configure the different stages of a software release process. CodePipeline automates the steps required to release your software changes continuously

Pipelines consist of **stages**. Movement between stages can be automatic or require manual approval. When a stage completes successfully, fails, or is canceled, an event is published to EventBridge.

Artifacts can be passed into stages and stages can result in an artifact as output.

Stages consist of one or more **actions** (single, sequential, or parallel).

<img src="./static/images/cicd_pipeline.png" alt="Code Pipeline - Conceptual" width="400"/>   
<img src="./static/images/codepipeline_arch.png" alt="Code Pipeline - Architecture" width="400"/>

Pipelines can be outfitted with a **manual approval** stage. In order for the current pipeline to continue, manual approval is required. If manual approval is not obtained within 7 days, the pipeline execution fails.

## CodePipeline Troubleshooting

**CodePipeline is not able to access a GitHub repository.**

CodePipeline uses GitHub OAuth tokens and personal access tokens to access your GitHub repositories.

- Check that no permissioned were revoked on the OAuth token in GitHub.

# CodeCommit

**CodeCommit** is a Git repository hosted by AWS.

Developers can clone repos using HTTPS (credentials), SSH (using a key-pair), or SSH (GRP). Developers are authorized to specific repositories using IAM roles.

In order to use SSH for CodeCommit, you must upload a public key for the user in IAM:  
<img src="./static/images/codecommit_iamssh.png" alt="Code Commit - Notifications" width="400"/>

CodeCommit supports notifications based on Git hooks. Notifications can be sent to SNS or AWS ChatBot (Slack).

<img src="./static/images/codecommit_notificationactions.png" alt="Code Commit - Notifications" width="400"/>

CodeCommit also support SNS and Lambda triggers based on supported events.

<img src="./static/images/codecommit_triggerevents.png" alt="Code Commit - Triggers" width="400"/>

CodeCommit can integrate with **Amazon CodeGuru**. Amazon CodeGuru is a developer tool that provides intelligent recommendations to improve code quality and identify an application’s most expensive lines of code. CodeGuru Reviewer uses machine learning and automated reasoning to identify critical issues, security vulnerabilities, and hard-to-find bugs during application development and provides recommendations to improve code quality.

> **Preventing passwords and other secrets from being committed to CodeCommit**     
> `git-secrets` scans commits, commit messages, and --no-ff merges to prevent adding secrets into your git repositories. If a commit, commit message, or any commit in a --no-ff merge history matches one of your configured prohibited regular expression patterns, then the commit is rejected.
> Setup `git-secrets` as a pre-commit hook to prevent secrets or passwords from being checked in.

# CodeBuild

CodeBuild is a fully-managed code build as a service product. Customers pay only for the resources consumed during the build. CodeBuild can be used to build and test code as part of a CI/CD pipeline.

By default, CodeBuild uses Docker for the build environment. The image can be customized.

CodeBuild code is sourced from GitHub, CodeCommit, CodePipeline, and S3.

Builds are defined as **Build Projects**.

<img src="./static/images/codebuild_arch.png" alt="Code Build - Architecture" width="400"/>

The `buildspec.yaml` (or json) is a file that can be used to customize builds through CodeBuild. `buildspec.yaml` must be in the root of the project.

[Build Spec Documentation](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html)

The buildspec defines four main phases of a build:
- `install`: install packages into the build environment (e.g., frameworks)
- `pre_build`: Sign into services and install code dependencies
- `build`: commands are run during the build process
- `post_build`: create artifacts/packages, push docker images, explicit notifications

The `buildspec.yaml` file can define environment variables that are available in the build process.

The `buildspec.yaml` file allows developers to customize artifact creation and storage.

CodeBuild supports **triggers**. Triggers allow developers to schedule builds based on Cron.

## CodeBuild Phases

<img src="./static/images/CodeBuild_Phase_Transitions.png" alt="Code Build - Phases" width="200"/>

# CodeDeploy

CodeDeploy can deploy applications to EC2, Lambda, AWS Elastic Beanstalk, AWS OpsWorks, CloudFormation, ECS (blue/green), Service Catalog, Alexa Skills Kit, and S3. It can also deploy to on-prem servers.

When deploying to EC2 or on-prem servers, the **CodeDeploy agent** must be installed.

Within CodeDeploy, developers create *Applications*. You can specify one or more *deployment groups* for a CodeDeploy application. The deployment group contains settings and configurations used during the deployment.

The deployment group requires a *service role* with permissions to deploy the application.

The following configurations are available to a deployment group:
- Deployment type (i.e., in-place or blue/green)
- Optionally specify an ASG or Load Balancer
- Deployment settings customize how the deployment operates, including:
    - Reroute traffic immediately or choose when to reroute traffic by specifing a wait period
    - Terminate or keep instances that have been replaced
    - Choose a default or custom deployment strategy
        - EC2 default deployment strategies include `CodeDeployDefault.AllAtOnce`, `CodeDeployDefault.HalfAtATime`, and `CodeDeployDefault.OneAtATime`
        - ECS/Lambda default deployment strategies include CodeDeployDefault.[Lambda,ECS]AllAtOnce, Linear, Canary
- Deployment triggers: CodeDeploy can publish up to 10 notifications to SNS based on lifecycle events (e.g., Deployment starts, Deployment succeeds, Deployment fails). Then, when that event occurs, all subscribers to the associated topic receive notifications through the endpoint specified in the topic, such as an SMS message or email message.
- Alarm configuration: The CodeDeploy deployment group will monitor up to 10 CloudWatch alarms. If the Alarm enters ALARM status, the deployment is stopped.
- Rollback configuration: The Deployment Group can be configured to roll back when a deployment fails, Roll back when alarm thresholds are met, or disable roll backs.
- Deployment group tags

If configured, CodeDeploy will install the AWS CodeDeploy agent on new instances through Systems Manager. The instances must already have the Systems Manager agent installed.

Custom Deployment Configurations can be created within CodeDeploy. 
  - When creating a new Deployment Configuration for Lambda or ECS, developers can choose between linear or canary deployments and customize the step and interval in which traffic is routed.
  - When creating a new Deployment Configuration for EC2/On-prem, developers can specify the minimum number or percentage of healthy EC2 instances that must be available at any time during the deployment.

The `appspec.yaml` (for json) is a file that can be used to customize deployments through CodeDeploy. `appspec.yaml` allows developers to define build configuration and lifecycle hooks for a deployment.

In the configuration section of `appspec.yaml`, there are three sections:
- `Files` (applies to EC2/On-prem servers only): provides information to CodeDeploy as to which files should be installed on the target server.
- `Permissions` (applies to EC2/On-prem servers only): details any special permissions and how they should be configured for the files in the `Files` section
- `Resources` (applies to ECS/Lambda only): defines metadata related to the Lambda function or ECS task

## CodeDeploy Lifecycle Hooks

The `appspec.yaml` configuration file supports lifecycle hooks during a deployment. The supported lifecycle hooks depend on the target (e.g., EC2/ECS/Lambda).

| Lifecycle Phase | Description | Can be scripted? |
| --- | --- | --- |
| ApplicationStop | This deployment lifecycle event occurs even before the application revision is downloaded. You can specify scripts for this event to gracefully stop the application or remove currently installed packages in preparation for a deployment. An AppSpec file does not exist on an instance before you deploy to it. For this reason, the ApplicationStop hook does not run the first time you deploy to the instance. | Y |
| DownloadBundle | During this deployment lifecycle event, the CodeDeploy agent copies the application revision files to a temporary location. | Y |
| BeforeInstall | You can use this deployment lifecycle event for preinstall tasks, such as decrypting files and creating a backup of the current version. | Y |
| Install | During this deployment lifecycle event, the CodeDeploy agent copies the revision files from the temporary location to the final destination folder. | N |
| AfterInstall | You can use this deployment lifecycle event for tasks such as configuring your application or changing file permissions. | Y |
| ApplicationStart | You typically use this deployment lifecycle event to restart services that were stopped during ApplicationStop. | Y |
| ValidateService | Can be used to verify the deployment was completed successfully. | Y |
| BeforeBlockTraffic | You can use this deployment lifecycle event to run tasks on instances before they are deregistered from a load balancer. | Y |
| BlockTraffic | During this deployment lifecycle event, internet traffic is blocked from accessing instances that are currently serving traffic. | N |
| AfterBlockTraffic | You can use this deployment lifecycle event to run tasks on instances after they are deregistered from a load balancer. | Y |
| BeforeAllowTraffic | You can use this deployment lifecycle event to run tasks on instances before they are registered with a load balancer. | Y |
| AllowTraffic | During this deployment lifecycle event, internet traffic is allowed to access instances after a deployment. | N |
| AfterAllowTraffic | You can use this deployment lifecycle event to run tasks on instances after they are registered with a load balancer. | Y |


Use the `Hooks` section of `appspec.yaml` to specify a Lambda function that CodeDeploy can call to validate a deployment. The deployment will fail if CodeDeploy is not notified by the Lambda function within one hour.

*Caption (below): Example of appspec.yaml Hooks section.*
```yaml
Hooks:
  - BeforeInstall: "BeforeInstallHookFunctionName"
  - AfterInstall: "AfterInstallHookFunctionName"
  - AfterAllowTestTraffic: "AfterAllowTestTrafficHookFunctionName"
  - BeforeAllowTraffic: "BeforeAllowTrafficHookFunctionName"
  - AfterAllowTraffic: "AfterAllowTrafficHookFunctionName"
```

*Caption (below): Example of a Lambda function hook. Notice how the `putLifecycleEventHookExecutionStatus` action is used, with the `deploymentId` and `lifecycleEventHookExecutionId` to report a success or failure.*
```Javascript
'use strict';

const aws = require('aws-sdk');
const codedeploy = new aws.CodeDeploy({apiVersion: '2014-10-06'});

exports.handler = (event, context, callback) => {
    //Read the DeploymentId from the event payload.
    var deploymentId = event.DeploymentId;

    //Read the LifecycleEventHookExecutionId from the event payload
    var lifecycleEventHookExecutionId = event.LifecycleEventHookExecutionId;

    /*
     Enter validation tests here.
    */

    // Prepare the validation test results with the deploymentId and
    // the lifecycleEventHookExecutionId for CodeDeploy.
    var params = {
        deploymentId: deploymentId,
        lifecycleEventHookExecutionId: lifecycleEventHookExecutionId,
        status: 'Succeeded' // status can be 'Succeeded' or 'Failed'
    };
    
    // Pass CodeDeploy the prepared validation test results.
    codedeploy.putLifecycleEventHookExecutionStatus(params, function(err, data) {
        if (err) {
            // Validation failed.
            callback('Validation test failed');
        } else {
            // Validation succeeded.
            callback(null, 'Validation test succeeded');
        }
    });
};
```

[AppSpec Documentation](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html)

## CodeDeploy Across Accounts

CodeDeploy does not natively support cross-account deployments. Instead, engineers must setup a CodeDeploy project in the other account and assume a role to interact with it.

# ECR

The Elastic Container Registry (ECR) is a managed container image registry service (similar to DockerHub). Each AWS account has a public and private registry.

Each registry consists of one or more repositories. Each repository contains many images. Images contain a tag. The image name and tag must be unique within a registry.

ECR is integrated with IAM permissions for authorization.

Basic or enhanced static image scanning is available using Inspector.

ECR supports cross-region and cross-account replication.
