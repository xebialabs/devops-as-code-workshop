# DevOps as Code workshop

This workshop will teach you:

* How to start up the XL DevOps Platform with the devops-as-code features installed
* How to install the XL CLI
* How to import and deploy a Docker application with XL Deploy
* How to import and run a pipeline with XL Release
* How to export XL YAML files to learn about the syntax

## Prerequisites

You'll need to have Docker installed on your machine before you begin:
* Mac: https://docs.docker.com/docker-for-mac/
* Windows: https://docs.docker.com/docker-for-windows/
* Linux: Refer to the instructions for your Linux distribution on how to install Docker

For part II of the workshop, you will also need:
* Access to an AWS account with administrator privileges
* Python

## Security warning

The docker compose setup potentially exposes weakly protected services to the network. Please use a firewall to block incoming network connections and avoid running the setup while connected to public networks. Also make sure to stop all containers by running `docker-compose down` when finished with the workshop. 

# Install the workshop

## Get the workshop

1) Download and extract the workshop zip:
```
$ curl -LO https://github.com/xebialabs/devops-as-code-workshop/archive/master.zip
$ unzip master.zip
$ cd devops-as-code-workshop-master
```
## Install licenses for XL Deploy and XL Release

The workshop requires you to bring your own licenses for XL Deploy and XL Release. You can use production licenses or request trial licenses for [XL Deploy](https://xebialabs.com/products/xl-deploy/trial/) and  [XL Release](https://xebialabs.com/products/xl-release/trial/).

1) Copy the XL Deploy license to `docker/xl-deploy/default-conf/deployit-license.lic`

2) Copy the XL Release license to `docker/xl-release/default-conf/xl-release-license.lic`

## Start up the XL DevOps Platform

1) If you are already running XL Deploy or XL Release on your local machine, please stop them.

2) If you are running the workshop on Windows, execute the following command to be able to run the Docker Compose file:

```
> set COMPOSE_CONVERT_WINDOWS_PATHS=1
```

For more information on this environment variable, read [the documentation for Docker Compose](https://docs.docker.com/compose/reference/envvars/#compose_convert_windows_paths)

3) Start up the XL DevOps Platform:

```
$ docker-compose up --build
```

4) Wait for XL Deploy and XL Release to have started up. This will have occurred when the following line is shown in the logs:
```
devops-as-code-workshop_xl-cli_1 exited with code 0
```

5) Open the XL Deploy GUI at http://localhost:4516/ and login with the username `admin` and password `admin`. Verify that the about box reports the version to be **8.5.0-alpha.25**.

6) Open the XL Release GUI at http://localhost:5516/ and login with the username `admin` and password `admin`. Verify that the about box reports the version to be **8.5.0-alpha.22**.

## Install the XL CLI

1) Open a new terminal window and install the XL command line client:

### Mac
```
$ curl -LO https://s3.amazonaws.com/xl-cli/bin/8.5.0-beta.1/darwin-amd64/xl
$ chmod +x xl
$ sudo mv xl /usr/local/bin
```

### Linux
```
$ curl -LO https://s3.amazonaws.com/xl-cli/bin/8.5.0-beta.1/linux-amd64/xl
$ chmod +x xl
$ sudo mv xl /usr/local/bin
```

### Windows
Download https://s3.amazonaws.com/xl-cli/bin/8.5.0-beta.1/windows-amd64/xl.exe
and place it somewhere on your `%PATH%`


2) Test the CLI by running the following the following command:
```
$ xl help
```

The output should look something like this:
```
XL Cli 8.5.0-beta.1
The xl command line tool provides a fast and straightforward method for provisioning
XL Release and XL Deploy with YAML files. The files can include items like
releases, pipelines, applications and target environments.

Usage:
  xl [command]

Available Commands:
  apply       Apply configuration changes
  export      Export configuration
  help        Help about any command
  version     Display version info

Flags:
      --config string                config file (default: $HOME/.xebialabs/config.yaml)
  -h, --help                         help for xl
  -q, --quiet                        suppress all output, except for errors
  -v, --verbose                      verbose output
      --xl-deploy-password string    Password to access the XL Deploy server (default "admin")
      --xl-deploy-url string         URL to access the XL Deploy server (default "http://localhost:4516/")
      --xl-deploy-username string    Username to access the XL Deploy server (default "admin")
      --xl-release-password string   Password to access the XL Release server (default "admin")
      --xl-release-url string        URL to access the XL Release server (default "http://localhost:5516/")
      --xl-release-username string   Username to access the XL Release server (default "admin")

Use "xl [command] --help" for more information about a command.
```

# Part I - Deploying a Docker application

## Exercise 1: Review the XL DevOps Platform running on Docker

When the XL DevOps Platform was started up by [the Docker Compose file](https://github.com/xebialabs/devops-as-code-workshop/blob/master/docker-compose.yaml), four containers were started:
* `xl-deploy` runs XL Deploy.
* `xl-release` runs XL Release.
* `dockerproxy` is where XL Deploy will deploy to. It's a proxy for the Docker engine on your local machine, the same that runs XL Release and XL Deploy..
* `xl-cli` runs the XL CLI to apply the [`configure-xl-devops-platform.yaml`](https://github.com/xebialabs/devops-as-code-workshop/blob/master/config/configure-xl-devops-platform.yaml) YAML file. This XL YAML file adds two configurations:
  * It adds an XL Deploy configuration to XL Release so that the latter can find the former.
  * It adds a `docker.Engine` configuration to XL Deploy so that XL Deploy can deploy to the Docker engine (via the Docker proxy).

1) Open the XL Deploy GUI, find the **local-docker** entry in the Infrastructure tree and run the **Check Connection** control task.


## Exercise 2: Set up the environment

Now that we have the XL DevOps Platform up and running and connected to our local Docker instance, let's create an environment that contains it.

1) Open a new terminal window and go to the directory where you unzipped the `devops-as-code-workshop` repository.

2) Create the environment that contains the Docker engine by applying its XL YAML file:
```
$ xl apply -f exercise-2/docker-environment.yaml
```
3) Open the XL Deploy GUI and review the environment that you just created.

## Exercise 3: Import a simple package and deploy it

Let's try and deploy something to our local Docker instance with XL Deploy. We'll start with a single container; the backend part of the REST-o-rant application. It's called **rest-o-rant-api** because it serves up the API.

1) Import the REST-o-rant-api package:

```
$ xl apply -f exercise-3/rest-o-rant-api-docker.yaml
```

2) Open the XL Deploy GUI and review version **1.0** of the **rest-o-rant-api-docker** application. Compare it with the contents of the XL YAML file that you applied in the previous step.

3) Using the GUI, deploy version **1.0** of the **rest-o-rant-api-docker** application to the **Local Docker Engine** environment.

4) When the deployment has finished, check whether the **rest-o-rant-api** container is running:
```
$ docker ps
```

The output should look something like this:
```
$ docker ps
CONTAINER ID        IMAGE                                           COMMAND                  CREATED             STATUS              PORTS                    NAMES
b0611ff9ffbb        xebialabsunsupported/rest-o-rant-api            "java -Djava.securit…"   11 seconds ago      Up 10 seconds       8080/tcp                 rest-o-rant-api
f80c2b00a88e        xebialabsunsupported/xl-release:8.5.0           "/opt/xebialabs/tini…"   3 days ago          Up About an hour    0.0.0.0:5516->5516/tcp   devops-as-code-workshop_xl-release_1
a99c31ddd458        tecnativa/docker-socket-proxy:latest            "/docker-entrypoint.…"   3 days ago          Up About an hour    2375/tcp                 devops-as-code-workshop_dockerproxy_1
68a5c6439540        xebialabsunsupported/xl-deploy:8.5.0            "/opt/xebialabs/tini…"   3 days ago          Up About an hour    0.0.0.0:4516->4516/tcp   devops-as-code-workshop_xl-deploy_1
```

## Exercise 4: Deploy a (slightly) more complex package

Serving a REST API is all nice and dandy, but it's pretty useless without a UI. So let's deploy the **rest-o-rant-web** container. Because it needs to access the **rest-o-rant-api** application to get its data, we'll also need to define a Docker network to allow the two containers to communicate. Version **1.1** of the **rest-o-rant-api** application will define that network.

1) Import the new version of the **rest-o-rant-api-docker** package, as well as the **rest-o-rant-web-docker** package:
```
$ xl apply -f exercise-4/rest-o-rant-docker.yaml
```

2) Open the XL Deploy GUI and review the two packages that you just imported and compare them with the XL YAML file. Notice the net **rest-o-rant-network** deployable in the **rest-o-rant-api-docker** package.

3) Using the GUI, deploy version **1.1** of the **rest-o-rant-api-docker** package to the **Local Docker Engine** environment.

4) Using the GUI, deploy version **1.0** of the **rest-o-rant-web-docker** package to the **Local Docker Engine** environment.

5) When the deployment has finished, open a new browser tab and access it at http://localhost:8181/. You should see a text saying "Find the best restaurants near you!".
Type "Cow" in the search bar and click "Search" to find the "Old Red Cow" restaurant.

## Exercise 5: Import a pipeline

OK, we've deployed the application, but how do we get rid of it? Well, let's do that manually for one last time:

1) Undeploy the **rest-o-rant-web** application from the **Local Docker Engine** environment.

2) Undeploy the **rest-o-rant-api** application from the **Local Docker Engine** environment.



But let's make sure that you don't forget next time that you run this workshop. Let's create a simple pipeline.

3) Import that REST-o-rant pipeline:
```
$ xl apply -f exercise-5/rest-o-rant-docker-pipeline.yaml
```

4) Open the XL Release GUI, go to **Design** tab, click on the **REST-o-rant** folder and then go to the **Templates** tab. Review the **REST-o-rant on Docker** pipeline that you've just imported and compare it to the XL YAML file.

5) Start a new release from that template and follow the instructions.

## Exercise 6: Simplify the package and the pipeline by using dependencies

You'll have noticed that the pipeline has two separate steps to deploy and later undeploy the **rest-o-rant-api-docker** and the **rest-o-rant-web-docker** packages. If we make the frontend depend on the backend, we can simplify the pipeline _and_ ensure that the frontend is not deployed without the backend.

In this exercise we will make the necessary modification in the package and the pipeline and then teach you how to export them back to YAML.

1) In XL Deploy, open **Applications/rest-o-rant-web-docker/1.0** and add the following entry to the **Application Dependencies** map:

| Key | Value	|
|------------------------	|-----	|
| rest-o-rant-api-docker 	| 1.1 	|

and change **Undeploy Unused Dependencies** to `true`.

2) In XL Release, from the **REST-o-rant on Docker** pipeline template, remove the **Deploy REST-o-rant application backend** task from the first phase and the **Undeploy REST-o-rant application backend** step from the last phase.

3) Run the pipeline to verify that the application dependencies and the updated pipeline function correctly. When the pipeline has completed, go to the XL Deploy UI and check that the **rest-o-rant-api-docker** application was undeployed.

4) Export the XL YAML files for the changes:

```
$ xl export -s xl-deploy -p Applications/rest-o-rant-web-docker -f exercise-6/rest-o-rant-web-docker-with-dependencies.yaml
$ xl export -s xl-release -p REST-o-rant -f exercise-6/rest-o-rant-pipeline-with-dependencies.yaml
```

5) Review the exported XL YAML files and compare them with the originals from exercises 4 and 5 respectively.

## Exercise 7: Export an XL YAML file with artifacts

The XL YAML format also allows you to use artifacts. To see what that looks like, let's export the PetClinic sample package:

1) In the XL Deploy UI, import the **PetClinic-ear/1.0** sample package from the XL Deploy server.

2) Export the XL YAML file for the package you just imported to see how to specify artifacts:
```
$ xl export -s xl-deploy -p Applications/PetClinic-ear/1.0 -f exercise-7/petlinic-ear.yaml
```

3) Review the exported XL YAML file and the artifact.


# Part II - AWS (ECS and Fargate)

In Part II of the workshop we will show you how to use the XL DevOps Platform to provision container infrastructure into AWS with CloudFormation and then deploy a simple monolith application into it.

For this part of the workshop, you'll need access to an AWS account with administrator privileges and you'll need to have Python installed on your machine.

## Exercise 8: Import AWS credentials into XL Deploy

Let's start by connecting XL Deploy with your AWS account.

1) Install the AWS CLI as described here:
https://docs.aws.amazon.com/cli/latest/userguide/installing.html

2) Configure the AWS CLI as described here and select region eu-west-1 (Ireland):
https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration

    **N.B.:** If you have already been using the AWS CLI and have multiple profiles, please ensure the default profile is configured correctly.

3) If you do not have Python installed on your machine, install it.

4) Run the provided Python script to generate an XL YAML file with your credentials:
```
$ ./exercise-8/awsconfig2xld.py > exercise-8/AWSConfig.yaml
```

5) Review the generate XL YAML file.

6) Apply the generated XL YAML file:
```
$ xl apply -f exercise-8/AWSConfig.yaml
```
7) In XL Deploy, navigate to the **Infrastructure/aws** object in the Explorer tree and execute the "Check connection" task to verify that the connection could be made.

## Exercise 9: Create AWS infrastructure with CloudFormation

Now privision the infrastructure using XL Deploy and CloudFormation. We'll let XL Deploy manage and run the CloudFormation template for us and let CloudFormation create the actual resources.

1) Import the CloudFormation template and its metadata into XL Deploy:
```
xl apply -f exercise-9/ecommerce-infrastructure.yaml 
```

2) In XL Deploy, deploy version **1.0** of the **ecommerce-infrastructure** package to the **AWS** environment.

3) When the deployment has finished, go to the AWS Console, navigate to the CloudFormation service, and review the **ecommerce-cloudformation** stack and the resources it has created:

    * A VPC with routes, subnets, etc.
    * IAM roles and policies
    * A load balancer with a target group
    * An ECS cluster

## Exercise 10: Deploy a monolith application 

Now that the infrastructure has been provisioned, let's deploy the applicaiton on top of it.

1) Import the e-commerce application into XL Deploy:
```
$ xl apply -f exercise-10/ecommerce-application.yaml 
```

2) Import the e-commerce pipeline into XL Release:
```
$ xl apply -f exercise-10/ecommerce-pipeline.yaml
```

3) In XL Release, create a new release based on the **e-Commerce CD pipeline** template in the **e-Commerce** folder. **Do not** start it yet!

4) We've already provisioned the infrastructure in the previous exercise. So, delete the first task of the first phase (**Provision e-commerce infrastructure**). In day-to-day use, one would let XL Release handle the complete orchestration, from provisioning the infrastructure to the very end when the infrastructure is deprovisioned.

5) Start the release.

6) When the release gets to the **Verify application manually** task, open the task and follow the instructions to complete it.

7) Wait until the release has finished.

## Exercise 11: Shutting down the XL DevOps Platform

1) Shut down the XL DevOps Platform:

```
$ docker-compose down
```

Not only will this stop the XL DevOps Platform, it will also remove any data stored on it. Therefore you should make sure that all releases and deployments have finished and that you've undeployed any applications you've deployed with it before you shut down the XL DevOps Platform.

## Bonus exercise: create something new!

OK, that was cool and all. Now use the `xl export` command to learn more about the YAML format and build your own YAML file.
