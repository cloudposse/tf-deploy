# terraform-aws-cicd [![Latest Release](https://img.shields.io/github/release/cloudposse/terraform-aws-cicd.svg)](https://github.com/cloudposse/terraform-aws-cicd/releases/latest) [![Slack Community](https://slack.cloudposse.com/badge.svg)](https://slack.cloudposse.com)

[![README Header][readme_header_img]][readme_header_link]

[![Cloud Posse][logo]](https://cpco.io/homepage)

<!--




  ** DO NOT EDIT THIS FILE
  **
  ** This file was automatically generated by the `build-harness`.
  ** 1) Make all changes to `README.yaml`
  ** 2) Run `make init` (you only need to do this once)
  ** 3) Run`make readme` to rebuild this file.
  **
  ** (We maintain HUNDREDS of open source projects. This is how we maintain our sanity.)
  **





-->

Terraform module to create AWS [`CodePipeline`](https://aws.amazon.com/codepipeline/) with [`CodeBuild`](https://aws.amazon.com/codebuild/) for [`CI/CD`](https://en.wikipedia.org/wiki/CI/CD)

This module supports three use-cases:

1. **GitHub -> S3 (build artifact) -> Elastic Beanstalk (running application stack)**.
The module gets the code from a ``GitHub`` repository (public or private), builds it by executing the ``buildspec.yml`` file from the repository, pushes the built artifact to an S3 bucket,
and deploys the artifact to ``Elastic Beanstalk`` running one of the supported stacks (_e.g._ ``Java``, ``Go``, ``Node``, ``IIS``, ``Python``, ``Ruby``, etc.).
    - http://docs.aws.amazon.com/codebuild/latest/userguide/sample-maven-5m.html
    - http://docs.aws.amazon.com/codebuild/latest/userguide/sample-nodejs-hw.html
    - http://docs.aws.amazon.com/codebuild/latest/userguide/sample-go-hw.html


2. **GitHub -> ECR (Docker image) -> Elastic Beanstalk (running Docker stack)**.
The module gets the code from a ``GitHub`` repository, builds a ``Docker`` image from it by executing the ``buildspec.yml`` and ``Dockerfile`` files from the repository,
pushes the ``Docker`` image to an ``ECR`` repository, and deploys the ``Docker`` image to ``Elastic Beanstalk`` running ``Docker`` stack.
    - http://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html


3. **GitHub -> ECR (Docker image)**.
The module gets the code from a ``GitHub`` repository, builds a ``Docker`` image from it by executing the ``buildspec.yml`` and ``Dockerfile`` files from the repository,
and pushes the ``Docker`` image to an ``ECR`` repository. This is used when we want to build a ``Docker`` image from the code and push it to ``ECR`` without deploying to ``Elastic Beanstalk``.
To activate this mode, don't specify the ``app`` and ``env`` attributes for the module.
    - http://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html


---

This project is part of our comprehensive ["SweetOps"](https://cpco.io/sweetops) approach towards DevOps.
[<img align="right" title="Share via Email" src="https://docs.cloudposse.com/images/ionicons/ios-email-outline-2.0.1-16x16-999999.svg"/>][share_email]
[<img align="right" title="Share on Google+" src="https://docs.cloudposse.com/images/ionicons/social-googleplus-outline-2.0.1-16x16-999999.svg" />][share_googleplus]
[<img align="right" title="Share on Facebook" src="https://docs.cloudposse.com/images/ionicons/social-facebook-outline-2.0.1-16x16-999999.svg" />][share_facebook]
[<img align="right" title="Share on Reddit" src="https://docs.cloudposse.com/images/ionicons/social-reddit-outline-2.0.1-16x16-999999.svg" />][share_reddit]
[<img align="right" title="Share on LinkedIn" src="https://docs.cloudposse.com/images/ionicons/social-linkedin-outline-2.0.1-16x16-999999.svg" />][share_linkedin]
[<img align="right" title="Share on Twitter" src="https://docs.cloudposse.com/images/ionicons/social-twitter-outline-2.0.1-16x16-999999.svg" />][share_twitter]


[![Terraform Open Source Modules](https://docs.cloudposse.com/images/terraform-open-source-modules.svg)][terraform_modules]



It's 100% Open Source and licensed under the [APACHE2](LICENSE).







We literally have [*hundreds of terraform modules*][terraform_modules] that are Open Source and well-maintained. Check them out!







## Usage


**IMPORTANT:** The `master` branch is used in `source` just as an example. In your code, do not pin to `master` because there may be breaking changes between releases.
Instead pin to the release tag (e.g. `?ref=tags/x.y.z`) of one of our [latest releases](https://github.com/cloudposse/terraform-aws-cicd/releases).


Include this repository as a module in your existing terraform code:

```hcl
module "build" {
    source              = "git::https://github.com/cloudposse/terraform-aws-cicd.git?ref=master"
    namespace           = "eg"
    stage               = "staging"
    name                = "app"

    # Enable the pipeline creation
    enabled             = true

    # Elastic Beanstalk
    elastic_beanstalk_application_name = "<(Optional) Elastic Beanstalk application name>"
    elastic_beanstalk_environment_name = "<(Optional) Elastic Beanstalk environment name>"

    # Application repository on GitHub
    github_oauth_token  = "(Required) <GitHub Oauth Token with permissions to access private repositories>"
    repo_owner          = "<GitHub Organization or Person name>"
    repo_name           = "<GitHub repository name of the application to be built and deployed to Elastic Beanstalk>"
    branch              = "<Branch of the GitHub repository>"

    # http://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref.html
    # http://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html
    build_image         = "aws/codebuild/standard:2.0"
    build_compute_type  = "BUILD_GENERAL1_SMALL"

    # These attributes are optional, used as ENV variables when building Docker images and pushing them to ECR
    # For more info:
    # http://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html
    # https://www.terraform.io/docs/providers/aws/r/codebuild_project.html
    privileged_mode     = true
    region              = "us-east-1"
    aws_account_id      = "xxxxxxxxxx"
    image_repo_name     = "ecr-repo-name"
    image_tag           = "latest"

    # Optional extra environment variables
    environment_variables = [{
      name  = "JENKINS_URL"
      value = "https://jenkins.example.com"
    },
    {
      name  = "COMPANY_NAME"
      value = "Amazon"
    },
    {
      name = "TIME_ZONE"
      value = "Pacific/Auckland"
   }]
}
```




## Examples

### Example: GitHub, NodeJS, S3 and EB

This is an example to build a Node app, store the build artifact to an S3 bucket, and then deploy it to Elastic Beanstalk running ``Node`` stack


``buildspec.yml`` file

```yaml
version: 0.2

phases:
  install:
    commands:
      - echo Starting installation ...
  pre_build:
    commands:
      - echo Installing NPM dependencies...
      - npm install
  build:
    commands:
      - echo Build started on `date`
  post_build:
    commands:
      - echo Build completed on `date`
artifacts:
  files:
    - node_modules/**/*
    - public/**/*
    - routes/**/*
    - views/**/*
    - app.js
```

### Example: GitHub, NodeJS, Docker, ECR and EB

This is an example to build a ``Docker`` image for a Node app, push the ``Docker`` image to an ECR repository, and then deploy it to Elastic Beanstalk running ``Docker`` stack

``buildspec.yml`` file

```yaml
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - $(aws ecr get-login --region $AWS_REGION)
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...
      - docker build -t $IMAGE_REPO_NAME .
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker image to ECR...
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
artifacts:
  files:
    - '**/*'
```

``Dockefile``

```dockerfile
FROM node:latest

WORKDIR /usr/src/app

COPY package.json package-lock.json ./
RUN npm install
COPY . .

EXPOSE 8081
CMD [ "npm", "start" ]

```



<!-- markdownlint-disable -->
## Makefile Targets
```text
Available targets:

  help                                Help screen
  help/all                            Display help for all targets
  help/short                          This help short screen
  lint                                Lint terraform code

```
<!-- markdownlint-restore -->
## Requirements

| Name | Version |
|------|---------|
| terraform | ~> 0.12.0 |
| aws | ~> 2.0 |
| local | ~> 1.2 |
| null | ~> 2.0 |
| random | ~> 2.1 |
| template | ~> 2.0 |

## Providers

| Name | Version |
|------|---------|
| aws | ~> 2.0 |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| attributes | Additional attributes (e.g. `policy` or `role`) | `list(string)` | `[]` | no |
| aws\_account\_id | AWS Account ID. Used as CodeBuild ENV variable when building Docker images. [For more info](http://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html) | `string` | `""` | no |
| branch | Branch of the GitHub repository, _e.g._ `master` | `string` | n/a | yes |
| build\_compute\_type | `CodeBuild` instance size.  Possible values are:<pre>BUILD_GENERAL1_SMALL</pre><pre>BUILD_GENERAL1_MEDIUM</pre><pre>BUILD_GENERAL1_LARGE</pre> | `string` | `"BUILD_GENERAL1_SMALL"` | no |
| build\_image | Docker image for build environment, _e.g._ `aws/codebuild/standard:2.0` or `aws/codebuild/eb-nodejs-6.10.0-amazonlinux-64:4.0.0` | `string` | `"aws/codebuild/standard:2.0"` | no |
| buildspec | Declaration to use for building the project. [For more info](http://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html) | `string` | `""` | no |
| codebuild\_cache\_bucket\_suffix\_enabled | The cache bucket generates a random 13 character string to generate a unique bucket name. If set to false it uses terraform-null-label's id value | `bool` | `true` | no |
| delimiter | Delimiter to be used between `name`, `namespace`, `stage`, etc. | `string` | `"-"` | no |
| elastic\_beanstalk\_application\_name | Elastic Beanstalk application name. If not provided or set to empty string, the `Deploy` stage of the pipeline will not be created | `string` | `""` | no |
| elastic\_beanstalk\_environment\_name | Elastic Beanstalk environment name. If not provided or set to empty string, the `Deploy` stage of the pipeline will not be created | `string` | `""` | no |
| enabled | Enable `CodePipeline` creation | `bool` | `true` | no |
| environment\_variables | A list of maps, that contain both the key 'name' and the key 'value' to be used as additional environment variables for the build | <pre>list(object(<br>    {<br>      name  = string<br>      value = string<br>  }))</pre> | <pre>[<br>  {<br>    "name": "NO_ADDITIONAL_BUILD_VARS",<br>    "value": "TRUE"<br>  }<br>]</pre> | no |
| force\_destroy | Force destroy the CI/CD S3 bucket even if it's not empty | `bool` | `false` | no |
| github\_oauth\_token | GitHub Oauth Token | `string` | n/a | yes |
| image\_repo\_name | ECR repository name to store the Docker image built by this module. Used as CodeBuild ENV variable when building Docker images. [For more info](http://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html) | `string` | `"UNSET"` | no |
| image\_tag | Docker image tag in the ECR repository, e.g. 'latest'. Used as CodeBuild ENV variable when building Docker images. [For more info](http://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html) | `string` | `"latest"` | no |
| name | Solution name, e.g. 'app' or 'jenkins' | `string` | n/a | yes |
| namespace | Namespace, which could be your organization name, e.g. 'eg' or 'cp' | `string` | `""` | no |
| poll\_source\_changes | Periodically check the location of your source content and run the pipeline if changes are detected | `bool` | `true` | no |
| privileged\_mode | If set to true, enables running the Docker daemon inside a Docker container on the CodeBuild instance. Used when building Docker images | `bool` | `false` | no |
| region | AWS Region, e.g. `us-east-1`. Used as CodeBuild ENV variable when building Docker images. [For more info](http://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html) | `string` | `""` | no |
| repo\_name | GitHub repository name of the application to be built (and deployed to Elastic Beanstalk if configured) | `string` | n/a | yes |
| repo\_owner | GitHub Organization or Person name | `string` | n/a | yes |
| stage | Stage, e.g. 'prod', 'staging', 'dev', or 'test' | `string` | `""` | no |
| tags | Additional tags (e.g. `map('BusinessUnit', 'XYZ')` | `map(string)` | `{}` | no |

## Outputs

| Name | Description |
|------|-------------|
| codebuild\_badge\_url | The URL of the build badge when badge\_enabled is enabled |
| codebuild\_cache\_bucket\_arn | CodeBuild cache S3 bucket ARN |
| codebuild\_cache\_bucket\_name | CodeBuild cache S3 bucket name |
| codebuild\_project\_id | CodeBuild project ID |
| codebuild\_project\_name | CodeBuild project name |
| codebuild\_role\_arn | CodeBuild IAM Role ARN |
| codebuild\_role\_id | CodeBuild IAM Role ID |
| codepipeline\_arn | CodePipeline ARN |
| codepipeline\_id | CodePipeline ID |




## Help

**Got a question?** We got answers.

File a GitHub [issue](https://github.com/cloudposse/terraform-aws-cicd/issues), send us an [email][email] or join our [Slack Community][slack].

[![README Commercial Support][readme_commercial_support_img]][readme_commercial_support_link]

## DevOps Accelerator for Startups


We are a [**DevOps Accelerator**][commercial_support]. We'll help you build your cloud infrastructure from the ground up so you can own it. Then we'll show you how to operate it and stick around for as long as you need us.

[![Learn More](https://img.shields.io/badge/learn%20more-success.svg?style=for-the-badge)][commercial_support]

Work directly with our team of DevOps experts via email, slack, and video conferencing.

We deliver 10x the value for a fraction of the cost of a full-time engineer. Our track record is not even funny. If you want things done right and you need it done FAST, then we're your best bet.

- **Reference Architecture.** You'll get everything you need from the ground up built using 100% infrastructure as code.
- **Release Engineering.** You'll have end-to-end CI/CD with unlimited staging environments.
- **Site Reliability Engineering.** You'll have total visibility into your apps and microservices.
- **Security Baseline.** You'll have built-in governance with accountability and audit logs for all changes.
- **GitOps.** You'll be able to operate your infrastructure via Pull Requests.
- **Training.** You'll receive hands-on training so your team can operate what we build.
- **Questions.** You'll have a direct line of communication between our teams via a Shared Slack channel.
- **Troubleshooting.** You'll get help to triage when things aren't working.
- **Code Reviews.** You'll receive constructive feedback on Pull Requests.
- **Bug Fixes.** We'll rapidly work with you to fix any bugs in our projects.

## Slack Community

Join our [Open Source Community][slack] on Slack. It's **FREE** for everyone! Our "SweetOps" community is where you get to talk with others who share a similar vision for how to rollout and manage infrastructure. This is the best place to talk shop, ask questions, solicit feedback, and work together as a community to build totally *sweet* infrastructure.

## Discourse Forums

Participate in our [Discourse Forums][discourse]. Here you'll find answers to commonly asked questions. Most questions will be related to the enormous number of projects we support on our GitHub. Come here to collaborate on answers, find solutions, and get ideas about the products and services we value. It only takes a minute to get started! Just sign in with SSO using your GitHub account.

## Newsletter

Sign up for [our newsletter][newsletter] that covers everything on our technology radar.  Receive updates on what we're up to on GitHub as well as awesome new projects we discover.

## Office Hours

[Join us every Wednesday via Zoom][office_hours] for our weekly "Lunch & Learn" sessions. It's **FREE** for everyone!

[![zoom](https://img.cloudposse.com/fit-in/200x200/https://cloudposse.com/wp-content/uploads/2019/08/Powered-by-Zoom.png")][office_hours]

## Contributing

### Bug Reports & Feature Requests

Please use the [issue tracker](https://github.com/cloudposse/terraform-aws-cicd/issues) to report any bugs or file feature requests.

### Developing

If you are interested in being a contributor and want to get involved in developing this project or [help out](https://cpco.io/help-out) with our other projects, we would love to hear from you! Shoot us an [email][email].

In general, PRs are welcome. We follow the typical "fork-and-pull" Git workflow.

 1. **Fork** the repo on GitHub
 2. **Clone** the project to your own machine
 3. **Commit** changes to your own branch
 4. **Push** your work back up to your fork
 5. Submit a **Pull Request** so that we can review your changes

**NOTE:** Be sure to merge the latest changes from "upstream" before making a pull request!


## Copyright

Copyright © 2017-2020 [Cloud Posse, LLC](https://cpco.io/copyright)



## License

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

See [LICENSE](LICENSE) for full details.

```text
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
```









## Trademarks

All other trademarks referenced herein are the property of their respective owners.

## About

This project is maintained and funded by [Cloud Posse, LLC][website]. Like it? Please let us know by [leaving a testimonial][testimonial]!

[![Cloud Posse][logo]][website]

We're a [DevOps Professional Services][hire] company based in Los Angeles, CA. We ❤️  [Open Source Software][we_love_open_source].

We offer [paid support][commercial_support] on all of our projects.

Check out [our other projects][github], [follow us on twitter][twitter], [apply for a job][jobs], or [hire us][hire] to help with your cloud strategy and implementation.



### Contributors

|  [![Erik Osterman][osterman_avatar]][osterman_homepage]<br/>[Erik Osterman][osterman_homepage] | [![Igor Rodionov][goruha_avatar]][goruha_homepage]<br/>[Igor Rodionov][goruha_homepage] | [![Andriy Knysh][aknysh_avatar]][aknysh_homepage]<br/>[Andriy Knysh][aknysh_homepage] |
|---|---|---|

  [osterman_homepage]: https://github.com/osterman
  [osterman_avatar]: https://img.cloudposse.com/150x150/https://github.com/osterman.png
  [goruha_homepage]: https://github.com/goruha
  [goruha_avatar]: https://img.cloudposse.com/150x150/https://github.com/goruha.png
  [aknysh_homepage]: https://github.com/aknysh
  [aknysh_avatar]: https://img.cloudposse.com/150x150/https://github.com/aknysh.png

[![README Footer][readme_footer_img]][readme_footer_link]
[![Beacon][beacon]][website]

  [logo]: https://cloudposse.com/logo-300x69.svg
  [docs]: https://cpco.io/docs?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=docs
  [website]: https://cpco.io/homepage?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=website
  [github]: https://cpco.io/github?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=github
  [jobs]: https://cpco.io/jobs?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=jobs
  [hire]: https://cpco.io/hire?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=hire
  [slack]: https://cpco.io/slack?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=slack
  [linkedin]: https://cpco.io/linkedin?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=linkedin
  [twitter]: https://cpco.io/twitter?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=twitter
  [testimonial]: https://cpco.io/leave-testimonial?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=testimonial
  [office_hours]: https://cloudposse.com/office-hours?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=office_hours
  [newsletter]: https://cpco.io/newsletter?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=newsletter
  [discourse]: https://ask.sweetops.com/?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=discourse
  [email]: https://cpco.io/email?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=email
  [commercial_support]: https://cpco.io/commercial-support?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=commercial_support
  [we_love_open_source]: https://cpco.io/we-love-open-source?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=we_love_open_source
  [terraform_modules]: https://cpco.io/terraform-modules?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=terraform_modules
  [readme_header_img]: https://cloudposse.com/readme/header/img
  [readme_header_link]: https://cloudposse.com/readme/header/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=readme_header_link
  [readme_footer_img]: https://cloudposse.com/readme/footer/img
  [readme_footer_link]: https://cloudposse.com/readme/footer/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=readme_footer_link
  [readme_commercial_support_img]: https://cloudposse.com/readme/commercial-support/img
  [readme_commercial_support_link]: https://cloudposse.com/readme/commercial-support/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/terraform-aws-cicd&utm_content=readme_commercial_support_link
  [share_twitter]: https://twitter.com/intent/tweet/?text=terraform-aws-cicd&url=https://github.com/cloudposse/terraform-aws-cicd
  [share_linkedin]: https://www.linkedin.com/shareArticle?mini=true&title=terraform-aws-cicd&url=https://github.com/cloudposse/terraform-aws-cicd
  [share_reddit]: https://reddit.com/submit/?url=https://github.com/cloudposse/terraform-aws-cicd
  [share_facebook]: https://facebook.com/sharer/sharer.php?u=https://github.com/cloudposse/terraform-aws-cicd
  [share_googleplus]: https://plus.google.com/share?url=https://github.com/cloudposse/terraform-aws-cicd
  [share_email]: mailto:?subject=terraform-aws-cicd&body=https://github.com/cloudposse/terraform-aws-cicd
  [beacon]: https://ga-beacon.cloudposse.com/UA-76589703-4/cloudposse/terraform-aws-cicd?pixel&cs=github&cm=readme&an=terraform-aws-cicd
