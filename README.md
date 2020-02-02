*This lab is part of a series of guides from the [Network Automation and Tooling workshop series](https://github.com/sttrayno/Network-Automation-Tooling)*

# Infrastructure as Code with Terraform

Terraform is an increasingly popular open-source infrastructure as code software tool built by HashiCorp. It enables administrators to define and provision manage infrastructure across multiple cloud and datacenter resources. Terraform takes an infrastructure as code approach by using using a high-level configuration language known as Hashicorp Configuration Language or JSON to define the resources. Terraform differs from traditional configuraiton management tools such as Ansible as it is known for keeping state, once you define your desired state Terraform looks to build your infrastucture then records its current state and always looks to maintain the desired state the use specifies

While Terraform has been increasingly used in the cloud space to provision infrastructure such as VMWare, AWS and Azure, we're starting to see more and more usage of this with Cisco infrastructure with support today for ASA firewalls and Cisco ACI in the data centre (Application Centric Infrastructure). Within these exercises we'll look to focus on how Terraform can be used to configure ACI and provision resources in todays enterprise IT environment.

## Exercise 0 - Installing Terraform

Terraform is supported across Windows, Linux/Unix and MacOS. The downloads for the latest version can be found from [here.](https://www.terraform.io/downloads.html)
Terraform is distributed as a single binary. Install Terraform by unzipping it and moving it to a directory included in your system's executable PATH. On Linux and MacOS systems you can do this through the command `$PATH` For more information on the install I'd recommend [this handy guide](https://www.vasos-koupparis.com/terraform-getting-started-install/)

To verify Terraform is installed use the command `Terraform` in the shell. Terraform should return a list of available commands.

![](images/terraform-install.gif)

To run these exercises you will need an instance of ACI. dcloud.cisco.com has a couple of instances of ACI that you can reserve and use in this lab we'll be using the "Cisco ACI 4.1 Automation v1" demo. Alternatively you can also use your own instance of ACI if you have one available or use the DevNet sandbox, the only drawback of using the sandbox is you cannot get direct API access to the ACI sim and must go through an RDP client to get access, which makes getting started a bit more convuluted, but feel free to use the sandbox if you'd prefer.

## Exercise 1 - Dipping our toe in the water, creating our first resources on ACI with Terraform

In this exercise we're going to look at understanding an example terraform config file thats defines the resources we're looking to privision. In this case our configuration file looks to provision a tenant, bridge domain, subnet, application profile and a couple of example Endpoint groupss related to the application. These are common tasks you'd look to do anytime your deploying a new application. 

It is important to note that within a single terraform configuration file we can configure resources on ACI, VMWare, AWS, Azure or any other infrastructure which Terraform has a provider for. This makes Terraform a very important tool for orchestrating the multiple parts of infrastructure that currently exist within enterprise environments, we'll look to cover these more advanced deployments as we go through the exercises but for now let's master the basics.

First thing you'll want to do is navigate to the examples/exercise1 folder within this project and have a look at the config.tf file. Open the file and have a look inside.

```Note: All Terraform configuration files will have the suffix .tf```

First off in the file you'll notice that we define the username, password and url of the ACI instance in order that Terraform can authenticate with ACI. We can authenticate through using a private key or through username or password as shown here, storing credentials in plain text isn't recommended but we'll make an exception and use it here so we can get up and running quickly. We'll cover other methods of authentication in later exercises. 

```
#configure provider with your cisco aci credentials.
provider "aci" {
  # cisco-aci user name
  username = "admin"
  # cisco-aci password
  password = "C1sco12345"
  # cisco-aci url
  url      = "https://198.18.133.200"
  insecure = true
}
```

Following the declaration of the ACI credentials the rest of the file looks to define the resources, in this initial configuration file you can see it defines a Tenant called "terraform_tenant", a bridge domain associated to that tenant and an example subnet. It then define an application profile for our "demo_ap" and 3 endpoint groups for our database, web and logic tiers for our 3 tier application. Study the file and make sure you understand it. If you're looking to understand more of the capabilities we have available from the ACI provider refer to the excellent documentation which exists on the Terraform [website](https://www.terraform.io/docs/providers/aci/index.html)

Before we can apply our state we must initialise Terraform. This will examine our config files and install the necessary providers, in this case ACI, this can be done by using the ```Terraform init``` command.

![](images/terraform-init.gif)

After Terraform is initialised all commands should now work. The next stage is to examine the changes that will be required to our infrastructure to implement our config files do this through excecuting the ```Terraform plan``` command.

![](images/terraform-plan.gif)

Examine the output of the command and look to understand it, if it has been sucessful it should list everything outlined in our configuration file as the infrastructure as it stands shouldn't have any of the config already applied and no current state exists. Terraform also provides a handy summary of how many adds, changes and destroys it will carry out when the testplan is executed. For example this scenario should outline:

```Plan: 7 to add, 0 to change, 0 to destroy.```

All thats left to do now we're happy with our changes that will be applied is to run the ```Terraform apply``` command, this will now go out and make the changes to our infrastructure that's been defined.

![](images/terraform-apply.gif)

Give the execution a few moments to run, it will ask you to type "yes" to confirm the changes that are about to happen. When its done you should get a nice confirmation at the bottom that the apply has been complete with 7 resources added, 0 changed and 0 destroyed. You can see from the output below that the resources are created sequentially as outlined in the configuration file. This is particuarly important in ACI as certain resources will have dependancies on others.

```
aci_tenant.terraform_tenant: Creating...
aci_tenant.terraform_tenant: Creation complete after 2s [id=uni/tn-tenant_for_terraform]
aci_application_profile.terraform_app: Creating...
aci_bridge_domain.bd_for_subnet: Creating...
aci_application_profile.terraform_app: Creation complete after 0s [id=uni/tn-tenant_for_terraform/ap-demo_ap]
aci_application_epg.application_epg2: Creating...
aci_application_epg.application_epg3: Creating...
aci_application_epg.application_epg1: Creating...
aci_bridge_domain.bd_for_subnet: Creation complete after 2s [id=uni/tn-tenant_for_terraform/BD-bd_for_subnet]
aci_subnet.demosubnet: Creating...
aci_subnet.demosubnet: Creation complete after 1s [id=uni/tn-tenant_for_terraform/BD-bd_for_subnet/subnet-[10.0.3.28/27]]
aci_application_epg.application_epg2: Creation complete after 3s [id=uni/tn-tenant_for_terraform/ap-demo_ap/epg-web_epg]
aci_application_epg.application_epg1: Creation complete after 3s [id=uni/tn-tenant_for_terraform/ap-demo_ap/epg-db_epg]
aci_application_epg.application_epg3: Creation complete after 4s [id=uni/tn-tenant_for_terraform/ap-demo_ap/epg-log_epg]
```

Lets now verify in ACI that the resources we requested have been created by following the animation below.

![](images/ACI-check.gif)

Congratuations, you've just completed your first exercise on using Terraform!

## Exercise 2 - Using Terraform Cloud with ACI for collaborative and automated provisioning

After exercise 1 you should now have a good grasp on what Terraform is, how you can define your intent for the infrastructure and how it works with ACI. In this exercise we're going to build on this and leverage Terraform cloud to start automating more changes to our ACI fabric. As we did in the previous steps, we'll create Terraform config files to define how we want our network (tenants, bridge domains, IP addressing) and applications (apps, epgs and contracts) to look. This time we'll store our config in a version control system (in this case Github) to start with the idea of provisioning. If you're new to VCS's you might want to check out this [guide](https://github.com/GShuttleworth/Introduction-to-Source-Control)

In the following steps we'll use two separate config files to define our state, the idea of using multiple files is to allow different teams (in this case the network and DevOps teams) to make changes to our fabric and have them deployed to the network. As you can see from the graphic below.

![](images/terraform-cicd.gif)

As mentioned previously, during this lab we'll use Terraform Cloud, an increasingly popular way to use Terraform today. Terraform Cloud is an SaaS application that helps teams use Terraform together. It manages Terraform runs in a consistent and reliable environment, and includes ways to share state and secret data, access controls for approving changes to infrastructure, a private registry for sharing Terraform modules, detailed policy controls for governing the contents of Terraform configurations, and more. For more information on Terraform cloud please view the excellent documentation [here](https://www.terraform.io/docs/cloud/index.html). 

There are two ways in which Terraform Cloud can run, remote and local. Remote usies the entire Terraform cloud runtime environment which will monitor configuration files on a VCS and when theres a change and carry out the plan and apply within the Terraform Cloud environment, the advantage of this is simplicity and it is very easy to build our CI/CD pipeline. However to do this the ACI controller must be accessible from the internet which with enterprise IT systems isn't always possible.

> Note: you can use Terraform Enterprise to get an on-premise hosted version of Terraform cloud. This could be a viable option for some organisations, but does come at a cost.

In local mode the Terraform runtime will excecute on the local machine however we still have the tracking functionality of Terraform which allows a shared state across multiple users. The advantage of this is it allows an easy way to share state between multiple users, in our case the DevOps and Infrastructure teams who can both manage their invdivual parts of the Terraform config. This method is a little more complex to set up but gives more flexibility to the user in therms of the other tools that sit as part of the pipeline so theres tradeoffs from both methods. As we're using a dCloud environment which sites beside a VPN this will probably be the easiest way for yourself to do this lab, but I'll outline both methods here.

### Step 0 - Pre-requisites 

In this lab, we'll use GitHub as the Version Control System (VCS) for our workspace. In order to follow along, you'll need a GitHub Account.

Once you have a GitHub account, visit the [example repository](https://github.com/sttrayno/tfc-aci-example/tree/master) and use the "Fork" button at the top right of the page to create a fork of the repository into your account. Once you've done that use the `git clone` command to create a local copy on your host too.

`git clone https://github.com/sttrayno/tfc-aci-example.` - This is crucial if you're following approach 2

Next, if you don't already have a Terraform Cloud account, you can create one from [the Terraform Cloud application](https://app.terraform.io/):

![](images/terraform-signup.gif)

### Approach 1 - Terraform Cloud (Remote mode)

### Step 1

If you've just signed up with Terraform Cloud and created a new organization, the first page you'll see is the "New Workspace" page. You can also create a new Workspace by choosing "Workspaces" from the main menu, and then the "New Workspace" button.

On the "New Workspace page", select "GitHub -> GitHub.com" to continue. A new window should open asking you to authorize Terraform Cloud to your GitHub account. If you have not logged into GitHub recently, you may need to log in first.

Click the green "Authorize" button to connect Terraform Cloud to your GitHub account.

Next, you will see a list of your GitHub repositories. Choose the repository you forked in the first step. If you have a lot of GitHub repositories, you may need to filter the list to find the correct one.

On the final step, leave the workspace name and "Advanced options" unchanged, and click the purple "Create workspace" button to create the workspace.

It may take a few minutes for Terraform Cloud to connect to your GitHub repository. Once that's complete, you should see a notification that your configuration was uploaded successfully:

![](images/tfc-github.gif)

### Step 2 - Advanced workspace configuration and first run

When you've created your initial workspace you're then given two options, to configure your workspace variables for the config file or to queue a plan to start the deployment

For an example we've left the variable hostname in our config file just to explain the process of adding variables into our workspace. 

In order to add the variable either select the button "Configure Variables" or the "Variables" tab from the workspace screen. Add a variable "hostname" with your value, in our case "https://198.18.133.200" to point towards our APIC controller. Save the variable you've created. If you'd like to create more variables and customise your config file feel free to, but in this guide I just wanted to give you the basics in Terraform variables so you're familiar

![](images/tfc-variables.gif)

>As of time of writing the ACI provider for Terraform is not supported by version 0.12, therefore we need to run v0.11.14. To configure this on Terraform Cloud simple go to the settings tab > general and set the version to 0.11.14.

>From the settings you can also configure other parameters such as the Terraform Working Directory which is the directory from your repository which you will excecute Terraform commands in, by default this will use the root, please change this to /remote in order that TFC reads the correct set of Terraform config files.

![](images/tfc-version.gif)
 
Now we've set up our workspace correctly, the only thing left to do is plan and apply the config. Much as we did in exercise 1.

In Terrafrom Cloud a plan will be run anytime that the user defines or anytime that a change is detected within the VCS. As we're just doing this for the first time and no changes are being made to the Github we will run our own plan by pressing the "Queue Plan" button on the top right, give a reason and watch the plan being run. The proposed changes will then be stored in a queue for the next apply.

When the plan runs, exactly as the CLI version did. Terraform will display the proposed changes from the plan.

![](images/tfc-que.gif)

Terraform Cloud will then ask you to confirm and apply the changes proposed

### Approach 2 - Terraform Cloud (local mode)

As we mentioned in the introduction another approach is to simply use Terraform Cloud as the backend only and a place to store Terraform state. When we manage infrastructure with Terraform, a .tfstate file is maintained, when new resources is defined in
config a difference is ran during the plan stage and this is how Terraform decides what it needs to create or destroy. This statefile. While useful poses a challenge when working in a collaborative environment, where multiple teams are making changes to Terraform state (In our case the DevOps and Infrastructure teams).

To solve this problem it is recommended when using Terraform outside a test environment to keep a central state, there are many options for doing this one of the most popular by is Terraform Cloud. 

As we mentioned earlier, as our dCloud environment we're using for this lab sits behind a VPN full Terraform Cloud runtime isn't a viable option as theres no direct network access. Therefore in this exercise we will show how to use TFC as a way to keep central state and have mutliple teams making/applying changes to a Terraform config.

### Step 1 - Creating a workspace on Terraform Cloud

Just as we did in approach 1. If you've just signed up with Terraform Cloud and created a new organization, the first page you'll see is the "New Workspace" page. You can also create a new Workspace by choosing "Workspaces" from the main menu, and then the "New Workspace" button.

This time on the "New Workspace page", select "No VCS connection" and give the workspace a name

On the final step give the workspace a name, leave "Advanced options" unchanged, and click the purple "Create workspace" button to create the workspace.

Now your workspace has been created, go into "Settings > General" and set the workspace runtime to local - This means the plan and apply commands will run on your local machine. Save your changes and set the TFC window to one side.

![](images/tfc-workspace.gif)

### Step 2 - Configuring Terraform locally

Now we have our workspace we have to configure our Terraform CLI to use TFC as the backend in order to store our infrastructures state, thankfully this is quite easy. Simply create or edit a file called .terraformrc to provide your token. To create this token go to user settings > token as shown below and create a new token. Then add the below excerpt to your .terraformrc file which is stored either in your %APPDATA% directory on windows or the users home directory on MacOS/Linux. You may need to create the file as it isn't created by default.

![](images/tfc-token.gif)

```
credentials "app.terraform.io" {
  token = "<YOUR TOKEN KEY HERE>"
}
```

Once you've configured the credentials. All that is left to do is go into the config-terraform.tf which I've included as part of the repo and change the organization to your TFC username, this and the credentials will direct Terraform to your TFC backend.

Also make sure your workspace name below matches what you have configued in TFC. This is very important

```
terraform {
  backend "remote" {
    hostname = "app.terraform.io"
    organization = "sttrayno"

    workspaces {
      name = "tfc-aci-example-local"
    }
  }
}
```

As mentioned, we will not be using TFC to make the changes, this is going to be done locally on the machine and initiated by the user. More on that in the next step.

### Step 3 - Applying Terraform config

Now lets build our infrastructure, this part of the process is pretty similar to what we did in exercise 1

Before we begin we're assuming you have your tfc-aci-examples git repo cloned on your local machine. In this exercise this is acting as our single source of truth fo the network,

First navigate to the directory of that where your config files are stored on your local machine.

As we would when working with a config file for the first time run the ```Terraform init```to initialise your providers and in this instance our newly configured shared backend 

![](images/terraform-init.gif)

After Terraform is initialised all commands should now work. As always the next stage is to run the ```Terraform plan``` command and see what resources will be created

![](images/terraform-plan.gif)

Examine the output as you normally would and ensure it is as expected, you should be familiar with this by now:

All thats left to do now we're happy with our changes that will be applied is to run the ```Terraform apply``` command to push our changes to the infrastructure. Confirm with yes when prompted.

![](images/terraform-apply.gif)

Give the execution a few moments to run, 

Now finally 

Congratulations, you've now got an understanding of Terraform Cloud and how it can be used to create a more robust, collaborative way of using Terraform to build infrastructure. In further exercises we'll look to create a more complex pipeline with checking and testing built in. If you have any feedback or issues with this lab please raise an issue or get in touch with me.

