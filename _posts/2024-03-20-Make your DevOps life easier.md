---
layout: single
title: "'Make' your DevOps life easier"
---
One of the common tasks you need to set up in your CICD pipeline is to assume an AWS role or run some program/scripts /commands such as bash, cloudformation or terraform.

Usually you will find the typical recommendation is to use the abstracted tasks (for example Github actions, Azure DevOps tasks…etc) that are built in to your specific CICD pipeline tool.

There are three downsides from my experience if you take this approach:

1. Your CICD pipeline code is locked to the specific provider (AWS, Azure, Github, Atlassian…etc)
2. Reliance on provider creating the abstractions in the first place and only the options/parameters they have allowed for that particular action.
3. Increased development cycle time since most providers require you to push the code to the repo to test and iterate (AFAIK there are exception like [act](https://github.com/nektos/act) for Github)

Over time, I have slowly phased out from using the built in abstractions to adopting a Makefile driven approach when it comes to my CICD pipeline. IMO there are a few advantages to using Make:

1. Make is ubiquitous, 90% of the time you will find it installed on your default CICD image and virtual machines.
2. Make allows chained dependencies, this means you can reuse make targets by setting them as upstream tasks.
3. You can set functions in Make by using a combination of [multi-line variables](https://www.gnu.org/software/make/manual/html_node/Multi_002dLine.html) and bash [positional arguments](https://linuxcommand.org/lc3_wss0120.php).
4. IMO Makefiles are cleaner than having shell scripts, where you might need different shell scripts for different tasks.

Today I will be showing a pattern that I have been using and iterated on which has worked quite well for me. When you adopt this method for executing your CICD pipeline tasks, there is little difference between what you execute locally when you’re developing and what is executed in your CICD pipeline.

In this example, we want to deploy MWAA/Airflow in AWS and have the following setup in Azure DevOps:

- Service connection set up for the CICD user, this service connection contains the AWS Access and Secret key for the user in the Tooling/CICD account that has no permission apart from using AWS STS to assume various roles in different target accounts.
- Cloudformation IaC code for deploying MWAA/Airflow in AWS.

### Infrastructure Pipeline

For our example, we use Cloudformation as our IaC and create change-sets for deploying infrastructure changes.

1. We want to first create the Cloudformation change-set.
2. Display the change-set in our CICD pipeline.
3. We have a manual validation step to approve or reject the changes.
4. We execute the change-set if approved.
5. We wait for the change-set to complete before moving on to later downstream tasks.

{% raw %}
```YAML
name: $(TeamProject)_$(Build.DefinitionName)_$(SourceBranchName)_$(Date:yyyyMMdd)$(Rev:.r)_mwaa_infra_pipeline

pool:
  name: 'Azure Pipelines'
  vmImage: ubuntu-latest

trigger:
  batch: true
  branches:
    include:
      - feature/*
      - test/*
      - main

  paths:
    include:
      - .ado/*
      - cfn/*
      - config/*

variables:
  - name: CICD_AWS_CREDENTIALS
    ${{ if eq(variables['Build.SourceBranchName'], 'main') }}:
      value: sc-prod-cicd-automation-credentials
    ${{ elseif startsWith(variables['Build.SourceBranch'], 'refs/heads/test') }}:
      value: sc-test-cicd-automation-credentials
    ${{ elseif startsWith(variables['Build.SourceBranch'], 'refs/heads/feature') }}:
      value: sc-dev-cicd-automation-credentials
    ${{ else }}:
      value: error

  - name: AIRFLOW_AWS_CREDENTIALS
    ${{ if eq(variables['Build.SourceBranchName'], 'main') }}:
      value: sc-prod-cicd-airflow-credentials
    ${{ elseif startsWith(variables['Build.SourceBranch'], 'refs/heads/test') }}:
      value: sc-test-cicd-airflow-credentials
    ${{ elseif startsWith(variables['Build.SourceBranch'], 'refs/heads/feature') }}:
      value: sc-dev-cicd-airflow-credentials
    ${{ else }}:
      value: error

  - name: Env
    ${{ if eq(variables['Build.SourceBranchName'], 'main') }}:
      value: prod
    ${{ elseif startsWith(variables['Build.SourceBranch'], 'refs/heads/test') }}:
      value: test
    ${{ elseif startsWith(variables['Build.SourceBranch'], 'refs/heads/feature') }}:
      value: dev
    ${{ else }}:
      value: error

stages:
  - stage: mwaa_deploy
    displayName:  "mwaa changeset creation"
    jobs:
    - job: changeset_create
      displayName: 'Create Changeset'
      steps:
      - task: UsePythonVersion@0
        inputs:
          versionSpec: '3.10'
      - task: AWSShellScript@1
        displayName: 'create change set'
        inputs: 
          awsCredentials: '${{ variables.CICD_AWS_CREDENTIALS }}'
          regionName: 'ap-southeast-2'
          scriptType: 'inline'
          inlineScript: |
            export ENVIRONMENT=${{ variables.Env }}
            echo $ENVIRONMENT
            make create-mwaa-changeset
          failOnStderr: true

      - task: AWSShellScript@1
        displayName: 'show change set'
        inputs: 
          awsCredentials: '${{ variables.CICD_AWS_CREDENTIALS }}'
          regionName: 'ap-southeast-2'
          scriptType: 'inline'
          inlineScript: |
            export ENVIRONMENT=${{ variables.Env }}
            echo $ENVIRONMENT
            make show-mwaa-changeset
          failOnStderr: true
      
    - job: waitForValidation
      dependsOn: changeset_create
      condition: always()
      displayName: 'Manual validation...'
      pool: server    
      timeoutInMinutes: 1440 
      steps:   
      - task: ManualValidation@0
        timeoutInMinutes: 1440
        inputs:
          notifyUsers: ''
          instructions: 'Please approve CFN changeset...'
          onTimeout: 'reject'

    - job: mwaa_execute
      dependsOn: waitForValidation
      displayName: 'execute changeset'
      steps:
      - task: AWSShellScript@1
        displayName: 'execute change set'
        inputs: 
          awsCredentials: '${{ variables.CICD_AWS_CREDENTIALS }}'
          regionName: 'ap-southeast-2'
          scriptType: 'inline'
          inlineScript: |
            export ENVIRONMENT=${{ variables.Env }}
            echo $ENVIRONMENT
            make execute-mwaa-changeset
          failOnStderr: true

    - job: wait_stack_complete
      dependsOn: mwaa_execute
      displayName: 'wait for stack complete'
      steps:
      - task: AWSShellScript@1
        displayName: 'execute change set'
        inputs: 
          awsCredentials: '${{ variables.CICD_AWS_CREDENTIALS }}'
          regionName: 'ap-southeast-2'
          scriptType: 'inline'
          inlineScript: |
            export ENVIRONMENT=${{ variables.Env }}
            echo $ENVIRONMENT
            make wait-complete-mwaa-changeset
{% endraw %}
```
There is quite a bit happening here but if we look at the general pattern, every single task is just running the `AWSShellScript` task (we use this since it automates the AWS CLI installation but there is nothing stopping you from adding the installation as an additional Make target) and we export an environment variable depending on the branch we are on. For example, `feature/*` maps to the development environment, `test/*` maps to the staging/test environment and `main` maps to the production environment. This `ENVIRONMENT` environment variable is used by the Makefile to source the correct configuration, more detail on that in the Makefile section.

Each task is just running a Make target command, resulting in minimal drift between your development workflow and what gets actually executed on the CICD platform.

### Makefile
```Makefile
SHELL := /bin/bash

include ./scripts/helpers.mk

.ONESHELL:
.PHONY: *
STACK_NAME=mwaa
ENVIRONMENT ?= dev


create-mwaa-changeset:
	source ./envs/$(ENVIRONMENT)/infra-base.env
	source ./envs/$(ENVIRONMENT)/infra-mwaa.env
	
	$(MAKE) changeset-create

show-mwaa-changeset:
	source ./envs/$(ENVIRONMENT)/infra-base.env
	source ./envs/$(ENVIRONMENT)/infra-mwaa.env

	$(MAKE) changeset-show

execute-mwaa-changeset:
	source ./envs/$(ENVIRONMENT)/infra-base.env
	source ./envs/$(ENVIRONMENT)/infra-mwaa.env

	$(MAKE) changeset-execute

wait-complete-mwaa-changeset:
	source ./envs/$(ENVIRONMENT)/infra-base.env
	source ./envs/$(ENVIRONMENT)/infra-mwaa.env

	$(MAKE) changeset-wait

delete-mwaa-stack:
	source ./envs/$(ENVIRONMENT)/infra-base.env
	source ./envs/$(ENVIRONMENT)/infra-mwaa.env
	$(MAKE) delete-stack
```
### helpers.mk
```Makefile
.SILENT: *
.PHONY: *
.ONESHELL:

define set_aws_creds
	aws sts assume-role --role-arn arn:aws:iam::$${TARGET_AWS_ACCOUNT_ID}:role/$${TARGET_ROLE_NAME} \
		--region "$${REGION}" \
		--role-session-name ml-mwaa > temp_creds.json && \
	export AWS_ACCESS_KEY_ID=$$(jq -r '.Credentials.AccessKeyId' temp_creds.json) && \
	export AWS_SECRET_ACCESS_KEY=$$(jq -r '.Credentials.SecretAccessKey' temp_creds.json) && \
	export AWS_SESSION_TOKEN=$$(jq -r '.Credentials.SessionToken' temp_creds.json)
endef

.SILENT: changeset-create
changeset-create:
	-@$(set_aws_creds)
	
	aws s3 cp ./cfn/mwaa-provisioning/ s3://$${CFN_S3_BUCKET_NAME}/${STACK_NAME}/ --recursive

	aws cloudformation deploy \
		--stack-name ${STACK_NAME} \
		--template-file $${TEMPLATE_BODY} \
		--parameter-overrides file://$${PARAMETERS_FILE} \
		--s3-bucket $${CFN_S3_BUCKET_NAME} \
		--s3-prefix ${STACK_NAME} \
		--tags "Environment=$${TAG_ENVIRONMENT}" "Owner=Baz" "Team=Foo Bar" \
		--capabilities "CAPABILITY_IAM" "CAPABILITY_NAMED_IAM" \
		--no-execute-changeset
		
changeset-show:
	-@$(set_aws_creds)

	ChangeSetName=$$(aws cloudformation list-change-sets --stack-name ${STACK_NAME} --output text --query 'Summaries[0].ChangeSetName')
	echo $${ChangeSetName}

	@aws cloudformation describe-change-set --stack-name ${STACK_NAME} --change-set-name $${ChangeSetName} --output table

changeset-execute:
	-@$(set_aws_creds)

	ChangeSetName=$$(aws cloudformation list-change-sets --stack-name ${STACK_NAME} --output text --query 'Summaries[0].ChangeSetName')
	echo $${ChangeSetName}

	@aws cloudformation execute-change-set --stack-name ${STACK_NAME} --change-set-name $${ChangeSetName}

changeset-wait:
	-@$(set_aws_creds)

	echo "Waiting for stack to be created..."

	aws cloudformation wait stack-create-complete --stack-name ${STACK_NAME}

delete-stack:
	-@$(set_aws_creds)
	aws sts get-caller-identity

	aws cloudformation delete-stack --stack-name ${STACK_NAME}

```
Let’s go through the Makefile in detail.

```Makefile
include ./scripts/helpers.mk
```
Make allows us to include additional Makefiles using `include ./scripts/helpers.mk` , we use this to separate out the “functions” in an effort to avoid having to have one single huge Makefile.

```Makefile
STACK_NAME=mwaa
ENVIRONMENT ?= dev
```
In this line we set the `STACK_NAME` environment variable, this is more specific to Cloudformation where we can have separate independent infrastructure stacks deployed in parallel by using unique stacks.

The default `ENVIRONMENT` is set as `dev` if not explicitly set, this applies generally when doing development work and avoids having to leave instructions to the developer on what environment variables they need to source or set before they can start working.

```Makefile
source ./envs/$(ENVIRONMENT)/infra-base.env
source ./envs/$(ENVIRONMENT)/infra-mwaa.env
```

Where in the `envs` folder we have `dev, test and prod`  folders that contains `infra-base.env` and `infra-mwaa.env` with different parameter/config contents.

Let’s see what’s in one of these .env files:
**infra-base.env**
```Shell
# environment variables for development testing
export TARGET_AWS_ACCOUNT_ID=<AWS Account ID>
export TARGET_ACCOUNT_ROLE_SESSION_NAME=ado-ops-cicd-role
export TARGET_ROLE_NAME=<Target role name we want to assume>
export CFN_S3_BUCKET_NAME=<S3 bucket used to store cfn files>
export REGION=ap-southeast-2
export SESSION_DURATION=1800
export TAG_ENVIRONMENT=Development
```
The `infra-base.env`  file contains general infrastructure environment variables that applies to all Cloudformation deployments for that specific environment, for example the S3 bucket, AWS account ID and target role name.

**infra-mwaa.env**
```Shell
# environment variables for deploying mwaa s3 bucket
export TEMPLATE_BODY='cfn/mwaa-provisioning/main-stack.yaml'
export PARAMETERS_FILE='cfn/parameters/mwaa/dev-params.json'
```
The infra-mwaa.env file contains environment variables that are specific to our MWAA infra deployment, specifically the TEMPLATE_BODY and PARAMETERS_FILE environment variables can be swapped out to another file if we want to deploy a different Cloudformation stack with different parameter overrides for Cloudformation.

For managing our Cloudformation IaC deployments, these are the Make targets we are concerned with:
```Makefile
create-mwaa-changeset:
	source ./envs/$(ENVIRONMENT)/infra-base.env
	source ./envs/$(ENVIRONMENT)/infra-mwaa.env
	
	$(MAKE) changeset-create

show-mwaa-changeset:
	source ./envs/$(ENVIRONMENT)/infra-base.env
	source ./envs/$(ENVIRONMENT)/infra-mwaa.env

	$(MAKE) changeset-show

execute-mwaa-changeset:
	source ./envs/$(ENVIRONMENT)/infra-base.env
	source ./envs/$(ENVIRONMENT)/infra-mwaa.env

	$(MAKE) changeset-execute

wait-complete-mwaa-changeset:
	source ./envs/$(ENVIRONMENT)/infra-base.env
	source ./envs/$(ENVIRONMENT)/infra-mwaa.env

	$(MAKE) changeset-wait

delete-mwaa-stack:
	source ./envs/$(ENVIRONMENT)/infra-base.env
	source ./envs/$(ENVIRONMENT)/infra-mwaa.env
	$(MAKE) delete-stack
```
They all follow a similar pattern, sourcing the correct environment variables before it proceeds to the subsequent Make target to create/show/execute/wait or delete the Cloudformation change-set.

Now, let’s go into the four main Cloudformation Make targets: create, show, execute and wait.

**Create Change-Set**
```Makefile
.SILENT: changeset-create
changeset-create:
	-@$(set_aws_creds)
	
	aws s3 cp ./cfn/mwaa-provisioning/ s3://$${CFN_S3_BUCKET_NAME}/${STACK_NAME}/ --recursive

	aws cloudformation deploy \
		--stack-name ${STACK_NAME} \
		--template-file $${TEMPLATE_BODY} \
		--parameter-overrides file://$${PARAMETERS_FILE} \
		--s3-bucket $${CFN_S3_BUCKET_NAME} \
		--s3-prefix ${STACK_NAME} \
		--tags "Environment=$${TAG_ENVIRONMENT}" "Owner=Baz" "Team=Foo Bar" \
		--capabilities "CAPABILITY_IAM" "CAPABILITY_NAMED_IAM" \
		--no-execute-changeset
```
This target first sets up the environment with the AWS credentials using `set_aws_creds`, the `-@` characters silences and allows the execution to continue if it errors (we would like this behaviour since when we use SSO to obtain temporary credentials).

The next line then copies our Cloudformation IaC files into our IaC bucket.

Finally, we run `aws cloudformation deploy` with various variables and configs that we either set in the Makefile such as `STACK_NAME` or environment variables in the shell such as `TEMPLATE_BODY`, `PARAMETERS_FILE...` from `infra-base.env` and `infra-mwaa.env`.

**Show Change-Set**
```Makefile
changeset-show:
	-@$(set_aws_creds)

	ChangeSetName=$$(aws cloudformation list-change-sets --stack-name ${STACK_NAME} --output text --query 'Summaries[0].ChangeSetName')
	echo $${ChangeSetName}

	@aws cloudformation describe-change-set --stack-name ${STACK_NAME} --change-set-name $${ChangeSetName} --output table
```
We would like to display the change-set we created from the previous step and decide to either approve or reject it.

Similar to the previous , we first obtain temporary creds from AWS via `set_aws_creds` . 

The next step is getting the randomly generated change-set name from AWS , note that there are some downside to the current code implementation. The biggest issue is that we are fetching the latest ChangeSetName using the `query` CLI arg, the issue is that if we have multiple pipelines at the same time for the same `STACK_NAME` then this creates a race condition and none deterministic behaviour. 

Last step displays the changes that will be applied by the change-set.

**Execute Change-Set**
```Makefile
changeset-execute:
	-@$(set_aws_creds)

	ChangeSetName=$$(aws cloudformation list-change-sets --stack-name ${STACK_NAME} --output text --query 'Summaries[0].ChangeSetName')
	echo $${ChangeSetName}

	@aws cloudformation execute-change-set --stack-name ${STACK_NAME} --change-set-name $${ChangeSetName}
```
Similar pattern to the previous two tasks, we set credentials, fetch the latest changeset name then we execute it. 

This step is only executed in the CICD pipeline once a manual stage is approved.

**Wait for Change-Set complete**
```Makefile
changeset-wait:
	-@$(set_aws_creds)

	echo "Waiting for stack to be created..."

	aws cloudformation wait stack-create-complete --stack-name ${STACK_NAME}
```
This task is required since we have downstream tasks that require the infrastructure changes to complete (MWAA sometimes can take up to 30 mins for changes to be complete). We use the built in aws cloudformation wait stack-create-complete command to await our stack to complete.

### Assuming role and export temporary credentials
```Makefile
define set_aws_creds
	aws sts assume-role --role-arn arn:aws:iam::$${TARGET_AWS_ACCOUNT_ID}:role/$${TARGET_ROLE_NAME} \
		--region "$${REGION}" \
		--role-session-name ml-mwaa > temp_creds.json && \
	export AWS_ACCESS_KEY_ID=$$(jq -r '.Credentials.AccessKeyId' temp_creds.json) && \
	export AWS_SECRET_ACCESS_KEY=$$(jq -r '.Credentials.SecretAccessKey' temp_creds.json) && \
	export AWS_SESSION_TOKEN=$$(jq -r '.Credentials.SessionToken' temp_creds.json)
endef
```
You will notice that for every single change-set Make target, we have a `-@$(set_aws_creds)` command in the first line.

This is required since for our particular AWS setup, the users are created in a dedicated AWS account and only have permission to assume a role in specific target accounts. Since the Azure DevOps CI/CD pipeline assumes the user using the service connection, we have to use `aws sts assume-role` to assume the specific role we want, exporting out the temporary credentials before we are able to execute the `aws cloudformation` commands.

Now note that other providers such as Github provides much better method to authenticate to AWS using OIDC. See https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services but for all intents and purposes this method solves our problem by first assuming the role, redirecting the output to a `temp_creds.json` file and then using `jq` to parse the JSON and export the necessary AWS credentials into the environment.

### Conclusion
I hope this article has been useful in explaining some of the difficulties and downsides to using abstracted task actions provided by CICD platforms. We presented an alternative method driven by Make and environment variables,  the pattern is similar to the 3 Musketeers pattern but refined specifically for Cloudformation IaC and CICD pipelines.


---
#### References:
- https://github.com/nektos/act
- https://www.gnu.org/software/make/manual/html_node/Multi_002dLine.html
- https://linuxcommand.org/lc3_wss0120.php

