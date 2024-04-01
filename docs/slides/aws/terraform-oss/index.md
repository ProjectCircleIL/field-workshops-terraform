name: AWS-Terraform-Workshop
class: center
count: false
![:scale 60%](images/tf_aws.png)
<br><br>
# AWS Terraform Workshop
## Build AWS Resources with Infrastructure as Code
???
INSTRUCTOR GUIDE LINK: https://github.com/hashicorp/field-workshops-terraform/blob/main/instructor-guides/aws_intro_to_terraform_INSTRUCTOR_GUIDE.md

This slide presentation is stored as Markdown code, specifically using the RemarkJS engine to render it. All standard markdown tags are supported, and you can also use some HTML within this document.

If you need to change the look and feel of the slide deck just use the style.css and remark_settings.js files to suit your needs. The content in this file is picked up by index.html when the page is loaded.

HTML comments like this one will show up in the source code, but not in the slides or speaker notes.
--->

Welcome to the beginner's guide to Terraform on AWS. This slide deck is written entirely in Markdown language, which means you can make edits or additions, then submit a pull request to add your changes to the main branch. To make edits to the slide deck simply fork this repository, edit the Markdown files, and submit a pull request with your changes.

The Markdown content is contained in the docs/ subdirectories.

Here are some helpful keyboard shortcuts for the instructor or participant:

⬆ ⬇ ⬅ ➡ - Navigate back and forth
P         - Toggle presenter view
C         - Pop an external window for presentation

Instructor notes are included in plain text, narrative parts are in **bold**. You can use the narrative quotes or change them to suit your own presentation style.

---
layout: true

.footer[
- ![:scale 100%](https://hashicorp.github.io/field-workshops-assets/assets/logos/HashiCorp_Icon_Black.svg)
]

---
name: Introductions
# Introductions
- Omri Siri
- CEO 
- 20+ Years in IT and Devops
- VIM is my favorite text editor

???
Use this slide to introduce yourself, give a little bit of your background story, then go around the room and have all your participants introduce themselves.

The favorite text editor question is a good ice breaker, but perhaps more importantly it gives you an immediate gauge of how technical your users are.

**There are no wrong answers to this question. Unless you say Notepad. Friends don't let friends write code in Notepad.**

**If you don't have a favorite text editor, that's okay! Our cloud lab has Visual Studio Code preinstalled. VSC is a free programmer's text editor for Microsoft, and it has great Terraform support. Most of this workshop will be simply copying and pasting code, so if you're not a developer don't fret. Terraform is easy to learn and fun to work with.**

---
name: Table-of-Contents
# Table of Contents

1. Intro to Terraform & Demo
1. Terraform Basics<br>
👩‍🔬 **Lab - Setup and Basic Usage**
1. Terraform In Action: plan, apply, destroy
1. Organizing Your Terraform Code<br>
🧪 **Lab - Terraform in Action**<br>
1. Provision and Configure AWS Instances<br>
🔬 **Lab - Provisioning with Terraform**<br>
1. Manage and Change Infrastructure State<br>
⚗️ **Lab - Terraform Remote State**


???
This workshop should take roughly three hours to complete.

**Here is our agenda for today's training. The format is simple, you'll hear a lecture and view slides on each topic, then participate in a hands-on lab about that topic. We'll alternate between lecture and lab, with a couple of breaks thrown in.**

---
name: intro-to-terraform-demo
class: title
# Chapter 1
## Introduction to Terraform

???
We use the word chapter here, because the training should feel like a story unfolding. The instructor's job is to guide the learners through this interactive story.

---
name: How-to-Provision-an-AWS-Instance
# How to Provision an AWS Instance

Let's look at a few different ways you could provision a new AWS Instance. Before we start we'll need to gather some basic information including (but not limited to):

- Instance Name
- Operating System (Image)
- VM Size
- Geographical Location (Region)
- Security Groups

???
**Has anyone got experience using AWS? How do most of us normally get started? That's right, we log onto the AWS Console and start clicking around. All of the major cloud providers make this part really easy. You get your account, log on and start clicking buttons. Let's take a peek at what that looks like...**

We chose the AWS Console as a starting point because most AWS users will be familiar with it. From this familiar starting point we begin our journey...

---
name: AWS-Console-Provision
# Method 1: AWS Console (GUI)
![:scale 70%](images/aws_provision.png)

???
**This should look familiar if you've ever used AWS. From the EC2 Panel, you click on Instances > Launch Instance, and you'll see a list of different AWS Machine Images (AMIs) you can use to provision your Instance. Some of these are provided by AWS, others are provided by third parties in the marketplace. You either search or browse for the thing you need, and click on it. Easy.**

---
name: AWS-Console-Provision-2
# Method 1: AWS Portal (GUI)
![:scale 60%](images/aws_provision_2.png)

???
**Once you've chosen your AMI, you will fill in some more details - the instance type, the VPC you want it to launch in, any associated IAM roles you want to assign it, external storage, tags, security groups... it's a long list of options! The AWS console can be handy for spinning up individual VMs and dev or test environments. The good news is that it's really easy to spin up infrastructure this way. The bad news is that it doesn't scale, and chances are that nobody is keeping track of what got built.**

It's really easy to make a big mess of things if you simply give everyone a console account and turn them loose in the cloud environment.

---
name: CloudFormation-Templates
class: compact
# Method 2: CloudFormation Templates
```json
{
  "AWSTemplateFormatVersion" : "2010-09-09",

  "Description" : "AWS CloudFormation Sample Template EC2InstanceWithSecurityGroupSample: Create an Amazon EC2 instance running the Amazon Linux AMI. The AMI is chosen based on the region in which the stack is run. This example creates an EC2 security group for the instance to give you SSH access. **WARNING** This template creates an Amazon EC2 instance. You will be billed for the AWS resources used if you create a stack from this template.",

  "Parameters" : {
    "KeyName": {
      "Description" : "Name of an existing EC2 KeyPair to enable SSH access to the instance",
      "Type": "AWS::EC2::KeyPair::KeyName",
      "ConstraintDescription" : "must be the name of an existing EC2 KeyPair."
    },
  }
}
```

CloudFormation Templates provide a consistent and reliable way to provision AWS resources. JSON is easy for computers to read, but can be challenging for humans to edit and troubleshoot.

???
**Which brings us to method #2, CloudFormation Templates, also known as CFTs. Have any of you used CFTs? What's that experience like?**

**CFTs are written in JSON, which stands for JavaScript Object Notation. It is an open-standard format for transmitting data between computers. And don't get me wrong, JSON is great. If you happen to be a computer. Computers are really good at reading these files full of key-value pairs and lists.**

**The problem is that editing and maintaining huge JSON files is hard for humans. Because JSON is not a programming language, you'll end up writing a lot more lines of complex code that is hard to understand and change.**

**CloudFormation Templates - easy for computers to read, hard for humans to troubleshoot and maintain.**

We are not here to bash on CloudFormation Templates or any other JSON/YAML based provisioning tools. The simple fact is these data formats are not well suited for logical operations (if/then, for loops, etc.)

---
name: Provision-with-Terraform-2
# Method 3: Provision with Terraform
```terraform
resource aws_instance "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"

  tags = {
    Name = "HelloWorld"
  }
}
```
.center[Example Terraform code for building an AWS instance.]

???
**And finally we have option #3, Terraform. Terraform uses a Domain Specific Language, or DSL that is designed to be both human-friendly and machine-readable. This is an example snippet of Terraform code. Now watch as I flip back to the previous slide. Would you rather have to write and maintain this complex and messy JSON, or this simple, compact terraform code?**

Advance back to the previous slide to illustrate the difference between JSON and equivalent Terraform.

---
name: What-is-Terraform
# What is Terraform?
```terraform
resource aws_instance "catapp" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  tags = {
    Name = "${var.prefix}-meow"
  }
```
* Executable Documentation
* Human and machine readable
* Easy to learn
* Test, share, re-use, automate
* Works on all major cloud providers


???
**So what exactly _is_ Terraform? Terraform is the DNA of your hybrid infrastructure. Terraform code is written in HCL, or HashiCorp Config Language. It is the only programming language designed specifically for provisioning infrastructure on any platform.**

**Do any of you have a wiki or set of runbooks that contain provisioning instructions? Think for a moment about that wiki. Now I want you to imagine the date stamp, when was this thing last edited? Let's say it was four years ago. Do you think maybe something could have changed in the past four years?**

**It sounds funny but the truth is your wiki is the obituary of the last known state of your infrastructure. One of the main reasons to use terraform is because it is self-documenting. The code itself explains every step required to build this infrastructure, and therefore it is always up to date.**

---
name: IaC
# What is Infrastructure as Code?
<br><br><br>
.biglist[
Infrastructure as Code (IaC) is the process of managing and provisioning cloud infrastructure with machine-readable definition files.

**Think of it as executable documentation.**
]

???
**You might be thinking...why can't I just do this by hand? After all the AWS portal is really easy, and I can just stand up my infrastructure manually. Here's why:**

**Terraform ensures that when you build any type of infrastructure that it gets built correctly every single time, exactly the same way. Let's try a thought experiment. If I gave every single one of you the same build document and asked you to set up a server, I guarantee there will be differences in those machines when you hand them over. They may not be major differences, but over time these can pile up and cause all sorts of uncertainty and issues in your environment.**

**When you require hands on keyboards (or mice), and you start making changes and manual steps on your infrastructure, you've lost the automation battle before it even starts. Even a single manual step can slow down your delivery schedule, and introduce unnecessary risk and change to your environments.**

---
name: IaC2
# Infrastructure as Code Allows Us To...
???
**We sometimes call this philosophy 'Infrastructure as Code', or the practice of expressing all of our provisioning steps as machine-readable code and variables. This is also known as the...**

---
name: IaC2
# Infrastructure as Code Allows Us To...
* Provide a codified workflow to create infrastructure
???
**...codified workflow. When you code-ify all of your manual steps, you'll gain several advantages that allow you to provision faster, with more efficiency, while reducing risk.**


---
name: IaC2
# Infrastructure as Code Allows Us To...
* Provide a codified workflow to create infrastructure
* Change and update existing infrastructure
???
**One of the main benefits of IaC is the ability to change and update what you built. There are many tools that allow you to provision infrastructure. This is sometimes called 'Day 0' of operations. The real challenge is managing Day N. What happens when you need to alter the infrastructure you built? Maybe you need to destroy or recreate part or all of it? Are you prepared to maintain and care for this infrastructure, without causing any downtime? Because Terraform is a _stateful_ tool, it can help you keep track of your infrastructure and change it with minimal impact.**

---
name: IaC2
# Infrastructure as Code Allows Us To...
* Provide a codified workflow to create infrastructure
* Change and update existing infrastructure
* Safely test changes using **`terraform plan`** in dry run mode
???
**Do you remember that scene in the movie Jurassic Park, where Samuel L Jackson turns around and says 'hold onto your butts' as he pushes his untested code change into production? Every sysadmin has had that feeling at one time or another. I really hope this works...**

**What if instead we had a way to safely test every change that went into production with a dry run? What would actually happen if I ran this code right now? Terraform comes with a built in dry run mode that allows you to visualize exactly what would happen if you pushed the apply button right now. This is a valuable tool for sysadmins and operations teams who prize stability and uptime.**

**Unexpected changes in the terraform plan output can be investigated _before_ they go into production.**

---
name: IaC2
# Infrastructure as Code Allows Us To...
* Provide a codified workflow to create infrastructure
* Change and update existing infrastructure
* Safely test changes using **`terraform plan`** in dry run mode
* Integrate with application code workflows (Git, CI/CD tools)

???
**Terraform allows you to automate manual processes and build continuous integration or continuous delivery pipelines. Imagine you had a pipeline for creating hardened machine images. Perhaps you have another pipeline for testing your infrastructure build process. These might be chained to other CI/CD application pipelines where the application is deployed into your tested, hardened infrastructure. Think of API driven infrastructure builds, written in a simple langage everybody can use and understand.**

---
name: IaC2
# Infrastructure as Code Allows Us To...
* Provide a codified workflow to create infrastructure
* Change and update existing infrastructure
* Safely test changes using **`terraform plan`** in dry run mode
* Integrate with application code workflows (Git, CI/CD tools)
* Provide reusable modules for easy sharing and collaboration

???
**As you expand your terraform usage, you'll have certain patterns and pieces of your infrastructure that you'd like to re-use. Maybe you want your network security to be set up a certain way, every time. Or perhaps someone wrote a great Terraform config for your web application. Terraform supports custom modules, which are simply packages of pre-built Terraform code that others can use. You can use Terraform modules to avoid repetition, enforce security, and ensure that standards are followed.**

---
name: IaC2
# Infrastructure as Code Allows Us To...
* Provide a codified workflow to create infrastructure
* Change and update existing infrastructure
* Safely test changes using **`terraform plan`** in dry run mode
* Integrate with application code workflows (Git, CI/CD tools)
* Provide reusable modules for easy sharing and collaboration
* Enforce security policy and organizational standards

???
**Terraform Enterprise also supports policy enforcement. You can create a list of dos and do-nots for your users and ensure that people don't build things they shouldn't, or introduce unnecessary risk into your environments. For example, you may have a policy that states that servers should not be exposed to the public internet. Because all your infrastructure is stored as code, you can quickly analyze that code to see if it's breaking any of the rules, preventing the bad behavior *before* the infrastructure gets built.**

---
name: IaC2
# Infrastructure as Code Allows Us To...
* Provide a codified workflow to create infrastructure
* Change and update existing infrastructure
* Safely test changes using **`terraform plan`** in dry run mode
* Integrate with application code workflows (Git, CI/CD tools)
* Provide reusable modules for easy sharing and collaboration
* Enforce security policy and organizational standards
* Enable collaboration between different teams

???
**Now that all your infrastructure is stored in a source code repository, it's very easy for multiple users and teams to collaborate on it. Developer needs a new feature? He or she can easily adjust the source code and send the change back to the operations folks for review. Terraform is a universal language that is understood by both developers and operations teams.**

---
name: IaC2
# Challenges of Infrastructure as Code
*State Management*: Managing state files can be complex, especially in large projects with multiple environments.
*Drift*: Infrastructure drift occurs when the actual state of your infrastructure deviates from the expected state defined in your code.
*Learning Curve*: The initial learning curve for IaC tools and principles can be steep for teams new to automation.

---
name: IaC-Tools
# Other Infrastructure as Code Tools
.center[![:scale 40%](images/infra_tools.png)]

These tools work well for configuring the operating system and application. They are not purpose-built for provisioning cloud infrastructure and platform services.

???
**Some of you might be thinking, that sounds great but what about this other tool that I use? Why shouldn't I just use Ansible since we already have that? Or my people only do Powershell. These are all great tools. But none of them are specifically designed for provisioning tasks.**

**Chef, Puppet and Ansible all work great in the context of your operating system and applications. It's true that you can do some cloud provisioning with each of these tools, but none of them really work as well as Terraform. And conversely, HashiCorp doesn't have a configuration management tool. Terraform works great with all of these tools.**

---
name: Native-Tools
# Native Cloud Provisioning Tools
.center[![:scale 90%](images/clouds.png)]

Each cloud has its own YAML or JSON based provisioning tool.

Terraform can be used across *all* major cloud providers and VM hypervisors.

???
**Every major cloud provider has their own JSON or YAML based provisioning tool. But all of them are written in YAML or JSON. And if you learn one of these systems, guess what, the others are completely different. Now if you want to have a multi-cloud strategy you have to learn three separate provisioning systems. With Terraform you are using the same language, the same simple syntax on all three of these cloud providers.**

---
name: Config-Hell
.center[![:scale 60%](images/Config_Hell.jpg)]
???
**This is a fun web comic. Those of you who have spent hours poking at a nested JSON template, trying to figure out which layer of curly braces you are in will understand this...**

---
Name: Terraform-vs-JSON
# Terraform vs. JSON
CFT JSON:
```json
"name": "{ "Fn::Join" : [ "-", [ PilotServerName, vm ] ] }",
```

Terraform:
```hcl
name = "${var.PilotServerName}-vm"
```

Terraform code (HCL) is easy to learn and easy to read. It is also 50-70% more compact than an equivalent JSON configuration.

---
Name: Why-Terraform-1
# Why Terraform?
.center[![:scale 60%](images/1password_terraform.png)]

.center[### <https://blog.1password.com/terraforming-1password/>]

???
1Password did a great blog post illustrating the difference between AWS Cloudformation (JSON) and Terraform.

https://blog.1password.com/terraforming-1password/

1Password were able to move their entire production infrastructure to Terraform in a few short weeks. Now they can tear down and completely rebuild their production environment in a matter of hours.

---
Name: Why-Terraform-on-AWS
# Why Terraform on AWS?

* Supports multi-cloud & hybrid infrastructure

???
**Why specifcially should you use Terraform on AWS? The first reason is that Terraform supports your hybrid or multi-cloud strategy. If you need to build some infrastructure on-prem, and some in AWS, Terraform is a natural fit. Your technical staff only has to learn a single language to be able to provision in either environment.**

---
Name: Why-Terraform-on-AWS
# Why Terraform on AWS?

* Supports multi-cloud & hybrid infrastructure
* Migrate from other cloud providers

???
**Terraform is also great for migrating between cloud providers. Let's say you wanted to move some workloads from AWS to AWS. The code changes in Terraform would be much easier to implement than they would via CloudFormation Templates. I was able to migrate a simple demo application from one cloud to another in a few short hours, because there was almost no learning curve. Terraform code looks the same no matter where you run it.**

---
Name: Why-Terraform-on-AWS
# Why Terraform on AWS?

* Supports multi-cloud & hybrid infrastructure
* Migrate from other cloud providers
* Increase provisioning speed

???
**It's not unusual to see provisioning times drop from days or weeks to hours or minutes when users adopt Terraform. Ineffective manual steps and change approvals can be replaced with fast code pipelines that have rigorous testing and security built right in. Now instead of waiting for days for a change request to be approved, users can self-provision their infrastructure without bottlenecks or slow approval processes.**

---
Name: Why-Terraform-on-AWS
# Why Terraform on AWS?

* Supports multi-cloud & hybrid infrastructure
* Migrate from other cloud providers
* Increase provisioning speed
* Improve efficiency

???
**Have you heard the saying 'measure twice, cut once?'? Terraform forces your operations teams to be disciplined and consistent with every single build. Have a change or setting that was overlooked during the build? Now you can immediately correct that mistake inside the code, so that a particular step never gets missed again. All future builds will contain the change. This can also improve relations between developers and operations, because the contract is clear. What gets built is always defined in the code, and never left to guesswork or manual processes.**

---
Name: Why-Terraform-on-AWS
# Why Terraform on AWS?

* Supports multi-cloud & hybrid infrastructure
* Migrate from other cloud providers
* Increase provisioning speed
* Improve efficiency
* Reduce risk

???
**Every modern IT organization has to deal with risk. It's a balancing act between security and usability. You can make it so secure nobody can use it, or on the other end you have a free for all where users can do whatever they want, but are putting the entire cloud account in jeopardy due to risky behavior. Terraform allows you to reduce risk by abstracting your users away from the web UI or API. Instead we provide a safe, auditable abstraction layer that lets users get their work done in a secure and safe way, that doesn't grant unnecessary privileged access.**

---
name: Live-Demo
class: title
# Live Demo

???
**Let's do a short demo! I'm going to show you how easy it can be to provision infrastructure in AWS. I'll do the demo on one of the lab workstations that you'll be using for this training.**

**This is a workstation just like the ones you'll be using for today's workshops. I'm going to run a terraform apply command to build out the lab environment. We're actually cheating a little bit here, as we prebaked most of the environment before class to save us some time. Just like your favorite cooking show!**

**You can see the results of the Terraform run here in my shell. This output is showing me the URL of the application server I just built. And if we pop over here to the AWS portal you'll see all of the different parts of my lab environment.**

**This is Infrastructure as code. By the end of today's training you'll be able to create your own infrastructure using Terraform.**

---

name: Chapter-2
class: title
# Chapter 2
## Terraform Basics

???
**Now that you have terraform installed and working with AWS, we can do a few dry runs before building real infrastructure. Follow along carefully, copying and pasting the commands on each slide into your shell as we go.**

---
name: what-is-terraform-oss
class: img-left
# What is Terraform?
![Terraform](images\Terraform_VerticalLogo_FullColor.png)

Terraform is an open source provisioning tool.

It ships as a single binary which is written in Go. Terraform is cross platform and can run on Linux, Windows, or MacOS.

Installing terraform is easy. You simply download a zip file, unzip it, and run it.

---
name: terraform-command-line
class: col-2
# Terraform Command Line
Terraform is a command line tool.

Terraform commands are either typed in manually or run automatically from a script.

The commands are the same whether you are on Linux or Windows or MacOS.

Terraform has subcommands that perform different actions.

```terraform
# Basic Terraform Commands
terraform version
terraform help
terraform init
terraform plan
terraform apply
terraform destroy
```

---
name: terraform-help
# Terraform Help
```tex
*$ terraform help
Usage: terraform [-version] [-help] <command> [args]
...
Common commands:
    apply              Builds or changes infrastructure
    console            Interactive console for Terraform interpolations
    destroy            Destroy Terraform-managed infrastructure
    env                Workspace management
    fmt                Rewrites config files to canonical format
    graph              Create a visual graph of Terraform resources
```
Type `terraform subcommand help` to view help on a particular subcommand.
???
**This is a good command to know. Terraform help will reveal all the available subcommands.**

---
name: terraform-code
# Terraform Code
```terraform
resource aws_vpc "main" {
  cidr_block       = "10.0.0.0/16"
  instance_tenancy = "dedicated"
}
```

Terraform code is based on the [HCL2 toolkit](https://github.com/hashicorp/hcl). HCL stands for HashiCorp Configuration Language.

Terraform code, or simply *terraform* is a declarative language that is specifically designed for provisioning infrastructure on any cloud or platform.

---
name: main.tf
# Terraform Comments
Line Comments begin with an octothorpe<sup>*</sup>, or pound symbol: #
```hcl
# This is a line comment.
```

Block comments are contained between /\* and \*/ symbols.
```tex
/* This is a block comment.
Block comments can span multiple lines.
The comment ends with this symbol: */
```
.small[
\* Yes, it really is called an [octothorpe](https://www.merriam-webster.com/dictionary/octothorpe).
]

---
name: terraform-workspaces
# Terraform Workspaces

A terraform workspace is simply a folder or directory that contains terraform code.

Terraform files always end in either a `*.tf` or `*.tfvars` extension.

Most terraform workspaces contain a minimum of three files:

**main.tf** - Most of your functional code will go here.<br>
**variables.tf** - This file is for storing variables.<br>
**outputs.tf** - Define what is shown at the end of a terraform run.<br>

---
name: terraform-init
# Terraform Init
```tex
*$ terraform init

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "aws" (hashicorp/aws) 2.35.0...
...
 provider.aws: version = "~> 2.35"

Terraform has been successfully initialized!

```
Terraform fetches any required providers and modules and stores them in the .terraform directory. If you add, change or update your modules or providers you will need to run init again.

???
**Terraform has an extendible architecture. You download the core program, terraform, then it fetches plugins and modules that are required for your code.**

---
name: terraform-plan
# Terraform Plan
```tex
*$ terraform plan
An execution plan has been generated and is shown below.
Terraform will perform the following actions:
  # aws_vpc.main will be created
  + resource "aws_vpc" "main" {
      + arn                              = (known after apply)
      + cidr_block                       = "10.0.0.0/16"
      ...
      + instance_tenancy                 = "dedicated"
    }
```
Preview your changes with `terraform plan` before you apply them.

???
**`terraform plan` is a dry run command. We're not actually building anything yet, Terraform is just telling is what it would do if we ran it for real.**

---
name: defining-variables
# Where are Variables Defined?
Terraform variables are placed in a file called *variables.tf*. Variables can have default settings. If you omit the default, the user will be prompted to enter a value. Here we are *declaring* the variables that we intend to use.

```tex
variable "prefix" {
  description = "This prefix will be included in the name of most resources."
}

variable "instance_tenancy" {
  description = "A tenancy option for instances launched into the VPC."
* default     = "dedicated"
}
```

???
**If you're curious where all these variables are defined, you can see them all in the _variables.tf_ file. Here we are simply defining all the available settings, and optionally declaring some default values. These defaults are what terraform will use if your user doesn't override them with their own settings.**

Q. Where could you override these defaults?<br>
A. In the terraform.tfvars file, or optionally on the command line or via environment variables. The most common approach is to use a tfvars file.

---
name: setting-variables
class: col-2
# How are Variables Set?
Once you have some variables defined, you can set and override them in different ways. Here is the level of precedence for each method.

This list goes from highest precedence (1) to lowest (5).

<br>
```tex
1. Command line flag - run as a
   command line switch
2. Configuration file - set in
   your terraform.tfvars file
3. Environment variable - part of
   your shell environment
4. Default Config - default value
   in variables.tf
5. User manual entry - if not
   specified, prompt the user
   for entry
```

---
name: lab-exercise-1
# 👩‍💻 Lab Exercise: Terraform Basics
<br><br>
In this lab you'll learn how to set up your editor, use the Terraform command line tool, integrate with AWS, and do a few dry runs with different settings.

# Terraform Basics Lab Exercise
## 👩‍💻 Get Hands-On with Terraform

---
# Objectives

1. Set Up Your Editor
2. Install Terraform
3. AWS Integration
4. First Terraform Configuration
5. Initialization and Planning
6. Applying Configuration
7. Destruction

---

# Step 1: Setting Up Your Editor

- Install the Terraform extension/plugin in your code editor for enhanced syntax highlighting and auto-completion support.

---

# Step 2: Installing Terraform

- Go to the [Terraform website](https://www.terraform.io/downloads.html) and download the appropriate package for your operating system.
- Follow the installation instructions to make Terraform available from your command line.

---

# Step 3: Configuring AWS Access

- Create an IAM user in your AWS account with programmatic access.
- Attach the `AmazonEC2FullAccess` policy.
- Configure your AWS CLI with the access key and secret key of the IAM user.

---

# Step 4: Writing Your First Terraform Configuration

```hcl
provider "aws" {
  region = "us-west-2"
}

resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-terraform-bucket"
  acl    = "private"
}
```
- Create a file named `main.tf`.
- Add the above content to define a provider and an S3 bucket resource.

---

# Step 5: Initializing and Planning

- Open your terminal and navigate to the directory containing `main.tf`.
- Run `terraform init` to initialize the workspace.
- Run `terraform plan` to see the proposed actions.

---

# Step 6: Applying the Configuration

- Execute `terraform apply` to create the S3 bucket.
- Confirm the action when prompted.

---

# Step 7: Destroying the Infrastructure

- Run `terraform destroy` to remove the resources you created.
- Confirm the action when prompted.

---

class: center, middle

# Congratulations!

## 🎉 You have completed the Terraform Basics Lab Exercise.

---
name: chapter-2-review
# 📝 Chapter 2 Review
.contents[
In this chapter we:
* Used the **`terraform init`** command
* Ran the **`terraform plan`** command
* Learned about variables
* Set our tenancy and prefix
]

---
name: Chapter-3
class: title
# Chapter 3
## Terraform in Action

???
**In this chapter we'll actually build real infrastructure using our sample code.**

---
name: anatomy-of-a-resource
# Anatomy of a Resource
Every terraform resource is structured exactly the same way.

```terraform
resource type "name" {
  parameter = "foo"
  parameter2 = "bar"
  list = ["one", "two", "three"]
}
```

**resource** = Top level keyword<br>
**type** = Type of resource. Example: `aws_instance`.<br>
**name** = Arbitrary name to refer to this resource. Used internally by terraform. This field *cannot* be a variable.

???
Everything else you want to configure within the resource is going to be sandwiched between the curly braces. These can include strings, lists, and maps.

---
name: provider-block
# Terraform Provider Configuration
The terraform core program requires at least one provider to build anything.

You can manually configure which version(s) of a provider you would like to use. If you leave this option out, Terraform will default to the latest available version of the provider.

```hcl
provider "aws" {
  version = "=2.35.0"
}
```

---
name: provider-versioning
# Versioning Operators

```tex
- = (or no operator): exact version equality
- !=: version not equal
- \>, >=, <, <=: version comparison
- ~>: pessimistic constraint, constraining both the oldest and newest
version allowed. ~> 0.9 is equivalent to >= 0.9, < 1.0, and ~> 0.8.4
is equivalent to >= 0.8.4, < 0.9
```
Re-usable modules should constrain only the minimum allowed version, such as >= 2.35.0.

???
**This specifies the earliest version that the module is compatible with while leaving the user of the module flexibility to upgrade to newer versions of Terraform without altering the module.**

---

name: terraform-apply
class: compact
# Terraform Apply
```tex
*$ terraform apply
An execution plan has been generated and is shown below.

Terraform will perform the following actions:
  # aws_vpc.main will be created
  + resource "aws_vpc" "main" {
      + cidr_block                       = "10.0.0.0/16"
      + instance_tenancy                 = "dedicated"
        ...
      + tags                             = {
          + "Name" = "main"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```
`terraform apply` runs a plan and then if you approve, it applies the changes.

---
name: terraform-destroy
class: compact
# Terraform Destroy
```tex
*$ terraform destroy
An execution plan has been generated and is shown below.

Terraform will perform the following actions:
  # aws_vpc.main will be destroyed
  - resource "aws_vpc" "main" {
      - cidr_block                       = "10.0.0.0/16" -> null
      - instance_tenancy                 = "dedicated" -> null
        ...
      - tags                             = {
          - "Name" = "main"
        } -> null
    }

Plan: 0 to add, 0 to change, 1 to destroy.
```
`terraform destroy` does the opposite. If you approve, your infrastructure is destroyed.
???
**Terraform can just as easily destroy infrastructure as create it. With great power comes great responsibility!**

---
name: terraform-fmt
# Terraform Format
Terraform comes with a built in code formatter/cleaner. It can make all your margins and list indentation neat and tidy. Beauty works better.

```tex
terraform fmt
```

Simply run it in a directory containing *.tf files and it will tidy up your code for you.

---

name: Data-Sources
class: compact
# Terraform Data Sources

```terraform
data "aws_ami" "ubuntu" {
  most_recent = true
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-trusty-14.04-amd64-server-*"]
  }
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  owners = ["099720109477"] # Canonical
}
```
Data sources are a way of querying a provider to return an existing resource, so that we can access its parameters for our own use.

---

name: dependency-mapping
class: compact
# Terraform Dependency Mapping
Terraform can automatically keep track of dependencies for you. Look at the two resources below. Note the highlighted lines in the aws_instance resource. This is how we tell one resource to refer to another in terraform.

```terraform
resource aws_key_pair "my-keypair" {
  key_name   = "my-keypair"
  public_key = file(var.public_key)
}

resource "aws_instance" "web" {
* ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
* key_name      = aws_key_pair.my-keypair.key_name
```
???
**Apart from the SSH keypair, you can also see how we reference the data source block from the previous slide. Flick back to the previous slide to show the relationship.**

---
name: organizing-your-terraform
# Organize Your Terraform Code
Terraform will read any file in your workspace that ends in a `.tf` extension, but the convention is to have a main.tf, variables.tf, and outputs.tf. You may add more tf files if you wish.

```bash
main.tf
variables.tf
outputs.tf
```

Let's take a closer look at each of these files.

---
name: terraform-main
class: compact
# The Main File

The first file is called main.tf. This is where you normally store your terraform code. With larger, more complex infrastructure you might break this up across several files.

```bash
# This is the main.tf file.
resource aws_vpc "main" {
  cidr_block       = var.cidr_block
  instance_tenancy = var.instance_tenancy
}

resource aws_subnet "main" {
  vpc_id     = aws_vpc.main.id
  cidr_block = var.cidr_block
  }
}
...
```

???
**We've removed all the comments from this code so it will fit on the slide.**

---
name: terraform-variables
class: compact
# The Variables File

The second file is called variables.tf. This is where you define your variables and optionally set some defaults.

```bash
variable "cidr_block" {
  description = "The address space that is used within the VPC. Changing this forces a new resource to be created."
}

variable "instance_tenancy" {
  description = "A tenancy option for instances launched into the VPC. Acceptable values are 'dedicated' and ''"
  default     = "dedicated"
}
```

---
name: terraform-outputs
class: compact
# The Outputs File
The outputs file is where you configure any messages or data you want to show at the end of a terraform apply.

```terraform
output "catapp_url" {
  value = "http://${aws_route53_record.hashicat.fqdn}"
}


output "private_key" {
  value = "${tls_private_key.hashicat.private_key_pem}"
}
```

???
**This bit here with the EOF is an example of a HEREDOC. It allows you store multi-line text in an output.**

---
name: tf-dependency-graph
class: img-right
# Terraform Dependency Graph
.center[![:scale 100%](images/blast_radius_graph_1.png)]

The terraform resource graph visually depicts dependencies between resources.

The region and prefix variables are required to create the resource group, which is in turn required to build the virtual network.

???
This is a good spot to talk a bit about how the dependency graph gets formed.

---
name: chapter-3-review
# 📝 Chapter 3 Review

In this chapter we:
* Learned about Terraform resources
* Ran terraform plan, graph, apply and destroy
* Learned about dependencies
* Viewed a graph of the lab
* Looked at main.tf, variables.tf and outputs.tf
* Built the Meow World application

---
name: Chapter-4
class: title
# Chapter 4
## Terraform State

---
name: terraform-state
class: compact
# Terraform State
Terraform is a _stateful_ application. This means that it keeps track of everything you build inside of a **state file**. You may have noticed the terraform.tfstate and terraform.tfstate.backup files that appeared inside your working directory. The state file is Terraform's source of record for everything it knows about.

```json
{
  "terraform_version": "0.12.7",
  "serial": 14,
  "lineage": "452b4191-89f6-db17-a3b1-4470dcb00607",
  "outputs": {
    "catapp_url": {
      "value": "http://go-hashicat-5c0265179ccda553.workshop.aws.hashidemos.io",
      "type": "string"
    },
  }
}
```

---
name: terraform-refresh
# Terraform Refresh

Sometimes infrastructure may be changed outside of Terraform's control.

The state file represents the *last known* state of the infrastructure. If you'd like to check and see if the state file still matches what you built, you can use the **terraform refresh** command.

Note that this does *not* update your infrastructure, it simply updates the state file.

```bash
terraform refresh
```

---
name: change-existing-infra
class: compact
# Changing Existing Infrastructure

Whenever you run a plan or apply, Terraform reconciles three different data sources:

1.  What you wrote in your code
2.  The state file
3.  What actually exists

Terraform does its best to add, delete, change, or replace existing resources based on what is in your *.tf files. Here are the four different things that can happen to each resource during a plan/apply:

```tex
+   create
-   destroy
-/+ replace
~   update in-place
```

---
name: state-quiz
class: compact
# Terraform State Quiz
| Configuration           | State                   | Reality                 | Operation |
| ----------------------- | ----------------------- | ----------------------- |:---------:|
| aws_instance |                         |                         |    ???    |
| aws_instance | aws_instance |                         |    ???    |
| aws_instance | aws_instance | aws_instance |    ???    |
|                         | aws_instance | aws_instance |    ???    |
|                         |                         | aws_instance |    ???    |
|                         | aws_instance |                         |    ???    |

What happens in each scenario? Discuss.

---
name: state-quiz-answers
class: compact
# Terraform State Quiz
| Configuration           | State                   | Reality                 | Operation    |
| ----------------------- | ----------------------- | ----------------------- |:------------:|
| aws_instance |                         |                         | create       |
| aws_instance | aws_instance |                         | create       |
| aws_instance | aws_instance | aws_instance | no-op        |
|                         | aws_instance | aws_instance | delete       |
|                         |                         | aws_instance | no-op        |
|                         | aws_instance |                         | update state |

What happens in each scenario? Discuss.

---
name: Chapter-5
class: title
# Chapter 5
## Resource dependencies and meta-arguments

---
name: Managing-Dependencies-and-Meta-Arguments-Intro
class: title
# Understanding Dependencies and Meta-Arguments

???
This module will introduce you to the concepts of managing dependencies and utilizing meta-arguments within your Terraform configurations. Mastering these concepts allows for more robust and reliable infrastructure as code.

---

name: Dependencies-Explanation
# Understanding Resource Dependencies

Terraform automatically determines dependencies between resources based on the configuration provided. Sometimes, explicit dependencies need to be defined to ensure resources are created in the correct order.

---

```hcl
resource "aws_security_group" "example" {
  // Security group configuration...
}

resource "aws_instance" "example" {
  // Instance configuration...

  vpc_security_group_ids = [aws_security_group.example.id]
}
```

In this example, Terraform understands that the instance must not be created until the security group is available.

---

name: Meta-Arguments-Explanation
# Understanding Meta-Arguments

Meta-arguments modify certain behaviors of resources. Common meta-arguments include `count`, `provider`, `lifecycle`, and `depends_on`.

---
```hcl
resource "aws_instance" "example" {
  count = 3 // Create three instances

  // Instance configuration...
}
```

This `count` meta-argument tells Terraform to create three instances based on this configuration.

---

name: Explicit-Dependencies
# Defining Explicit Dependencies with `depends_on`

Sometimes Terraform's implicit dependency resolution needs help. Use `depends_on` to explicitly define dependencies.

---

```hcl
resource "aws_s3_bucket" "example" {
  // Bucket configuration...
}

resource "aws_s3_bucket_object" "example" {
  // Object configuration...

  depends_on = [aws_s3_bucket.example]
}
```

This ensures the S3 bucket is created before the object.

---

name: Meta-Arguments-Lifecycle
# Using `lifecycle` Meta-Arguments

`lifecycle` meta-arguments control the lifecycle of a resource, such as preventing accidental deletion.

---

```hcl
resource "aws_s3_bucket" "example" {
  // Bucket configuration...

  lifecycle {
    prevent_destroy = true
    # This configuration prevents the bucket from being destroyed when applying `terraform destroy`.
    create_before_destroy = true
    # This configuration creates a new bucket before destroying the old one.
    ignore_changes = [tags]
    # This configuration ignores changes to the tags attribute.
  }
}
  }
}
```

---

name: Lab-Managing-Dependencies
class: title
# Lab Exercise: Managing Dependencies

In this lab, you'll practice defining explicit dependencies between resources and using meta-arguments to control resource behavior.

---
## Objective:
Create an AWS VPC, a security group within that VPC, and an instance that uses the security group. Ensure the security group is created before the instance.
---
### Steps:
1. **Define the VPC:**
   Create a `main.tf` with a VPC resource.
   ```hcl
   resource "aws_vpc" "example_vpc" {
     cidr_block = "10.0.0.0/16"
     tags = {
       Name = "example_vpc"
     }
   }
   ```
---
2. **Define the Security Group:**
   Add a security group that depends on the VPC.
   ```hcl
   resource "aws_security_group" "example_sg" {
     vpc_id = aws_vpc.example_vpc.id
     // Additional configuration...
   }
   ```
---
3. **Define the Instance:**
   Create an instance that uses the security group. Use the `depends_on` meta-argument to ensure the security group is created first.
   ```hcl
   resource "aws_instance" "example_instance" {
     ami = "ami-0c55b159cbfafe1f0"
     instance_type = "t2.micro"
     vpc_security_group_ids = [aws_security_group.example_sg.id]
     // Ensure the instance is created after the security group
     depends_on = [aws_security_group.example_sg]
   }
   ```
---
4. **Apply Configuration:**
   Initialize Terraform and apply your configuration. Observe the order in which resources are created.

---
### Deliverable:
Submit the `main.tf` file along with a screenshot of the `terraform apply` command output showing the resources being created in the correct order.


---

name: Importing-Infrastructure-Intro
class: title
# Mastering Infrastructure Import in Terraform

One of Terraform's powerful features is the ability to import existing infrastructure. This enables you to bring resources under Terraform's management without needing to recreate them.

---

name: How-to-Import
# Step-by-Step Guide to Importing Resources

Importing resources into Terraform involves identifying the existing resource in your infrastructure and mapping it to a Terraform resource in your configuration.

---
### Example: Importing an AWS S3 Bucket:
1. **Define the Resource in Terraform:**
   First, define a placeholder resource in your `main.tf` for the S3 bucket you intend to import.
   ```hcl
   resource "aws_s3_bucket" "imported_bucket" {
     # Intentionally left blank
   }
   ```
---
2. **Use the `terraform import` Command:**
   Use the `terraform import` command to map the existing S3 bucket to your Terraform resource.
   ```shell
   terraform import aws_s3_bucket.imported_bucket my-existing-bucket-name
   ```
---
3. **Review the State:**
   After importing, review your Terraform state to ensure the resource has been successfully imported and is now managed by Terraform.

---

name: Lab-Importing-Infrastructure
class: title
# Lab Exercise: Importing Existing Infrastructure

In this lab, you'll practice importing existing infrastructure into Terraform. You will import an existing AWS resource and bring it under Terraform management.

---
## Objective:
Import an existing AWS S3 bucket into Terraform.
---

### Steps:
1. **Identify an Existing S3 Bucket:**
   Choose an existing S3 bucket in your AWS account to import. Note its name.
---
2. **Prepare the Terraform Configuration:**
   Define a placeholder AWS S3 bucket resource in your `main.tf`.
---
3. **Import the S3 Bucket:**
   Use the `terraform import` command to import the S3 bucket into Terraform management.
---
4. **Verify the Import:**
   Review the Terraform state to confirm the S3 bucket is now managed by Terraform.
---

### Deliverable:
Submit your `main.tf` file and a screenshot of the `terraform state list` command output showing the imported S3 bucket.

---

name: Advanced-Import-Techniques
class: title
# Advanced Techniques for Terraform Import

Terraform's import feature is a powerful tool for bringing existing infrastructure under Terraform management. This section explores advanced techniques for leveraging import in complex scenarios and best practices for state manipulation.

---

name: Complex-Import-Scenarios
# Handling Complex Import Scenarios

Some resources or modules may require importing multiple components or related resources. Complex scenarios might involve resources that are interdependent or require a specific import order.
---

### Strategy for Complex Imports:
1. **Sequential Imports:** For resources with dependencies, import lower-level resources first, gradually moving up the dependency chain.
2. **Module Imports:** If using modules, consider importing the module's underlying resources individually, then refactoring your configuration to represent the module structure.
---

### Example: Importing AWS EC2 Instance with Attached Volumes
- Import the EC2 instance first:
  ```shell
  terraform import aws_instance.my_instance i-1234567890abcdef0
  ```
- Then, import each EBS volume attached to it:
  ```shell
  terraform import aws_ebs_volume.my_volume vol-049df61146f4d7901
  ```
- Finally, ensure your Terraform configuration reflects the relationship between the instance and volumes.

---

name: State-File-Manipulation
# Manipulating the Terraform State File

After importing resources, you may need to manipulate the Terraform state file to accurately reflect your infrastructure's desired configuration.
---

### Moving Resources within the State
The `terraform state mv` command allows you to move items within the state file, which is particularly useful after importing resources into a module structure.
---

```shell
terraform state mv aws_instance.my_instance module.my_module.aws_instance.my_instance
```
---

### Removing Items from the State
Use `terraform state rm` to remove items from the state that are no longer managed by Terraform.

```shell
terraform state rm aws_instance.unmanaged_instance
```

---

name: Refactoring-Post-Import
# Refactoring Configurations Post-Import

After importing resources, you'll likely need to refactor your Terraform configurations to align with your infrastructure's current state and organization.
---

### Steps for Refactoring:
1. **Review Imported Resources:** Examine the state file and the attributes of imported resources.
2. **Update Terraform Configurations:** Adjust your Terraform configurations to match the imported state, ensuring resource attributes and module structures are accurately represented.
3. **Validate and Plan:** Use `terraform plan` to identify any discrepancies between your configuration and the current state.

### Best Practices:
- **Incremental Changes:** Make and apply changes incrementally to minimize impact and risk.
- **Documentation:** Update documentation to reflect any changes in the infrastructure as code.

---

name: Lab-Advanced-Import-and-State-Manipulation
class: title
# Lab: Advanced Import and State Manipulation

This lab focuses on importing complex resources and manipulating the state file to integrate imported resources into your Terraform configurations effectively.
---

## Objectives:
1. Import a complex resource with dependencies.
2. Manipulate the Terraform state to reorganize resources post-import.
3. Refactor your Terraform configurations to align with the imported state.

---
### Steps:
1. **Select a Complex Resource:** Choose a resource with dependencies for import, such as an AWS EC2 instance with attached security groups and EBS volumes.
2. **Import Resources Sequentially:** Follow the strategy for complex imports to bring all components under Terraform management.
3. **Manipulate State File:** Use `terraform state mv` and `terraform state rm` as needed to organize your state file.
4. **Refactor Terraform Configurations:** Update your configurations to reflect the imported infrastructure accurately.
5. **Validate Changes:** Run `terraform plan` to ensure no configuration drifts and your infrastructure is accurately represented.

---
### Deliverable:
Submit a brief report detailing the resources you chose to import, the steps taken to manipulate the state file, and any challenges encountered during refactoring. Include before and after snippets of your Terraform configuration.

---

name: Securing-Terraform-Deployments-Intro
class: title
# Securing Terraform Deployments

Ensuring the security of your Terraform deployments involves managing secrets and sensitive data carefully. This section will cover strategies for securely handling secrets in Terraform.

---

name: Managing-Secrets
# Managing Secrets in Terraform

Terraform can use various methods to manage secrets and sensitive information, ensuring they are not exposed in your configurations or state files.
---

### Using Environment Variables:
Environment variables can be used to keep secrets out of your Terraform configurations.
```shell
export TF_VAR_secret_key="your_secret_key_here"
```
These can then be referenced in your Terraform configurations without hardcoding sensitive information.
---

### Using Terraform Vault Provider:
The Terraform Vault provider allows storing and retrieving secrets from HashiCorp Vault, a dedicated secrets management tool.
```hcl
data "vault_generic_secret" "db_password" {
  path = "secret/data/db/password"
}

output "db_password" {
  value = data.vault_generic_secret.db_password.data["password"]
  sensitive = true
}
```
---

### Marking Variables as Sensitive:
You can mark variables as sensitive in Terraform to prevent their values from being displayed in logs or console output.
```hcl
variable "api_key" {
  type      = string
  sensitive = true
}
```

---

name: Lab-Securing-Deployments
class: title
# Lab Exercise: Securing Terraform Deployments

In this lab, you will practice securing your Terraform deployments by managing secrets with environment variables and the Vault provider.

---
## Objective:
Configure a Terraform project to securely manage an API key.

---
### Steps:
1. **Set up an Environment Variable:**
   Define an environment variable for your API key.
2. **Reference the Environment Variable:**
   Modify your Terraform configuration to use the environment variable for any resource requiring the API key.
3. **Use Vault for Extra Security (Optional):**
   If you have access to a Vault server, store and retrieve the API key using the Vault provider in Terraform.
4. **Apply Your Configuration:**
   Run `terraform apply` and ensure the API key is used correctly without being exposed in logs.

---
### Deliverable:
Submit your modified Terraform configuration files and a brief explanation of the steps you took to secure the API key. Ensure no sensitive information is included in your submission.

---

name: Advanced-Terraform-Techniques-Intro
class: title
# Advanced Terraform Techniques

Beyond the basics, Terraform offers advanced techniques for more dynamic and flexible infrastructure management. This section explores loops, conditional expressions, and error handling.

---

name: Loops-and-Conditional-Expressions
# Using Loops and Conditional Expressions

Terraform supports loop constructs like `count`, `for_each`, and dynamic blocks to create multiple instances of resources based on a list or map. Conditional expressions can be used to include or modify resources based on specified conditions.
---

### Example: Creating Multiple Instances with `count`:
```hcl
resource "aws_instance" "app_server" {
  count         = length(var.instance_ids)
  ami           = var.ami_id
  instance_type = "t2.micro"
}
```

---
### Conditional Creation of Resources:
```hcl
resource "aws_elastic_ip" "eip" {
  count = var.create_eip ? 1 : 0
  instance = aws_instance.app_server[0].id
}
```
---

name: Advanced-Loops-in-Terraform
class: title
# Advanced Loops in Terraform with `for_each` and `for` Loops

Terraform's `for_each` and `for` constructs allow for more dynamic and flexible infrastructure management. Here, we explore several examples to understand how to effectively use these constructs.

---

name: For-Each-Usage
# Utilizing `for_each` with Resources

The `for_each` construct is used to create multiple instances of a resource based on a map or a set of strings.
---

### Example: Creating Multiple S3 Buckets
```hcl
variable "bucket_names" {
  type    = set(string)
  default = { "logs", "uploads", "backups" }
}

resource "aws_s3_bucket" "app_buckets" {
  for_each = var.bucket_names

  bucket = each.key
  acl    = "private"
}
```
This example creates three S3 buckets with names "logs", "uploads", and "backups". Each bucket is uniquely identified by its name within the `for_each` loop.

---

name: For-Loop-Expressions
# Using `for` Loops in Expressions

Terraform's `for` loops can be used within expressions to transform and filter data structures.
---

### Example: Transforming a List of Strings
```hcl
variable "instance_names" {
  type    = list(string)
  default = ["instance1", "instance2", "instance3"]
}

output "instance_tags" {
  value = { for name in var.instance_names : name => "${name}-tag" }
}
```
This `for` loop expression takes a list of instance names and transforms it into a map where each value is suffixed with "-tag".
---

### Filtering with `for` Loops:
```hcl
variable "numbers" {
  default = [1, 2, 3, 4, 5]
}

output "even_numbers" {
  value = [for n in var.numbers : n if n % 2 == 0]
}
```
This example filters the list of numbers to only include even numbers using a `for` loop with an `if` condition.

---

name: Dynamic-Block-For-Each
# Dynamic Blocks with `for_each`

Dynamic blocks can be used in conjunction with `for_each` to dynamically configure parts of a resource.
---

### Example: Creating Multiple Security Group Rules
```hcl
resource "aws_security_group" "app_sg" {
  name = "app_sg"

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value["from_port"]
      to_port     = ingress.value["to_port"]
      protocol    = ingress.value["protocol"]
      cidr_blocks = ingress.value["cidr_blocks"]
    }
  }
}
```
---
```hcl
variable "ingress_rules" {
  description = "A map of ingress rules for the security group"
  type = map(object({
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
  }))
```
---
```hcl
  default = {
    ssh = {
      from_port   = 22
      to_port     = 22
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    http = {
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```
This example demonstrates the use of dynamic blocks to create multiple ingress rules for a security group, with the rules defined in a variable.

---

name: Error-Handling
# Error Handling in Terraform

Error handling in Terraform can be managed through careful use of input variable validations, preconditions, and sensible defaults to prevent configuration errors.

---

### Example: Variable Validation
```hcl
variable "instance_type" {
  type        = string
  description = "EC2 instance type"

  validation {
    condition     = contains(["t2.micro", "t2.small", "t2.medium"], var.instance_type)
    error_message = "The instance type must be t2.micro, t2.small, or t2.medium."
  }
}
```

This validation ensures the `instance_type` variable is within the expected values, preventing potential runtime errors.

---

name: Lab-Advanced-Techniques
class: title
# Lab Exercise: Advanced Terraform Techniques

In this lab, you will apply advanced Terraform techniques by utilizing loops, conditional expressions, and implementing error handling in your configurations.
---

## Objective:
Enhance a Terraform configuration to dynamically create resources based on input variables and conditions, with proper error handling.
---

### Steps:
1. **Dynamic Resource Creation:**
   Use `count` or `for_each` to create a dynamic number of AWS S3 buckets based on input variables.
2. **Conditional Resource Creation:**
   Add a condition to create an AWS Elastic IP only if a specific variable is set to true.
3. **Add Variable Validation:**
   Implement validation for an input variable to ensure it meets specific criteria.

---

### Deliverable:
Submit your Terraform configuration files demonstrating the use of advanced techniques. Include a brief description of how each technique is applied in your configuration.

---

name: Introduction-to-Terraform-Modules
class: title
# Introduction to Terraform Modules

Terraform modules encapsulate a collection of resources that are used together. This section introduces the concept of modules, explaining why they're beneficial and how they can be utilized to create more maintainable and reusable Terraform configurations.
---

name: Understanding-Terraform-Modules
# Understanding Terraform Modules

A Terraform module is a container for multiple resources that are used together. Modules can be used to create lightweight abstractions, so that you can describe your infrastructure in terms of its architecture, rather than directly in terms of physical objects.
---

### Benefits of Using Modules:
- **Reusability:** Write once, use multiple times. Avoid duplicating code.
- **Organization:** Group related resources together to organize infrastructure components logically.
- **Manageability:** Simplify complex infrastructure into manageable chunks.
- **Versioning:** Utilize different versions of modules for different environments or updates.

---

name: Creating-Terraform-Modules
# Creating Terraform Modules

Creating a module involves defining resources in a dedicated directory. This directory can then be referenced from other Terraform configurations.
---

### Steps to Create a Module:
1. **Define the Module:** Create a new directory for your module and add `.tf` files that define the resources the module will manage.
2. **Use Input Variables:** Define variables for your module to accept input values.
3. **Define Output Values:** Specify outputs to return information about the resources in the module.

---
### Example: Creating a VPC Module
- **Directory Structure:**
  ```
  modules/
  └── vpc/
      ├── main.tf
      ├── variables.tf
      └── outputs.tf
  ```
---

- **main.tf:**
  ```hcl
  resource "aws_vpc" "example" {
    cidr_block = var.cidr_block
    tags = {
      Name = "MyVPC"
    }
  }
  ```

---
- **variables.tf:**
  ```hcl
  variable "cidr_block" {
    description = "The CIDR block for the VPC"
    type        = string
  }
  ```

---
- **outputs.tf:**
  ```hcl
  output "vpc_id" {
    value = aws_vpc.example.id
  }
  ```

---

name: Using-Terraform-Modules
# Using Terraform Modules

Once you have created a module, you can use it in your Terraform configurations by referencing its path.
---

### Example: Referencing a Local Module
```hcl
module "my_vpc" {
  source      = "./modules/vpc"
  cidr_block  = "10.0.0.0/16"
}
```
---

### Example: Referencing a Module from Terraform Registry
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "2.77.0"
  cidr_block  = "10.0.0.0/16"
  # Other required variables...
}
```

---

name: Refactoring-to-Modules-Lab
class: title
# Lab: Refactoring to Terraform Modules

This lab exercise guides you through the process of refactoring an existing Terraform configuration into a modular architecture.
---

## Objective:
Refactor an existing flat Terraform configuration into a more manageable, modular structure.
---

### Scenario:
You have a Terraform configuration that defines an AWS VPC, subnets, and security groups. Your task is to refactor this configuration into a modular structure.
---

### Steps:
1. **Analyze the Existing Configuration:** Identify logical groupings of resources that can be abstracted into modules (e.g., networking resources).
2. **Create Modules:** Based on your analysis, create separate directories for each module and move the related resources into the appropriate module.
3. **Define Module Inputs and Outputs:** For each module, define necessary input variables and outputs.
4. **Update the Main Configuration:** Replace the original resource definitions in your main configuration with references to your new modules.
5. **Test Your Refactoring:** Initialize and apply your configuration to ensure that the modular structure correctly replicates the original architecture without changes to the infrastructure.

---
### Deliverable:
Submit your refactored Terraform configuration, including the main configuration file and the created modules. Include a brief explanation of how you organized the resources into modules and any challenges you encountered.

---
name: Integrating-Terraform-into-CICD-Intro
class: title
# Integrating Terraform into CI/CD Pipelines

Automating infrastructure management with CI/CD pipelines enhances efficiency, reduces human error, and ensures consistency across environments. This section explores how to integrate Terraform into CI/CD workflows.
---

name: Terraform-CICD-Concepts
# Key Concepts for Terraform CI/CD Integration

Integrating Terraform into CI/CD pipelines involves several key practices:
---

### Version Control:
Ensure all Terraform configurations are stored in a version control system (VCS) to track changes and collaborate.
---

### Automated Testing:
Incorporate automated testing of Terraform configurations to catch errors early. Tools like `terraform validate` and `terraform plan` can be used in pipelines for this purpose.
---

### Remote State Management:
Use remote state backends such as Terraform Cloud or AWS S3 with state locking to manage state files securely and collaboratively.
---

### Infrastructure as Code (IaC) Workflow:
Adopt a consistent IaC workflow, utilizing pull requests for changes, code reviews, and automated deployments through the pipeline.
---

name: CI-CD-Tools-and-Terraform
# CI/CD Tools and Terraform
Several CI/CD tools can be used to automate Terraform workflows, including Jenkins, GitLab CI, GitHub Actions, and CircleCI. The choice of tool depends on your existing ecosystem and specific needs.
---

### Example: GitHub Actions Workflow for Terraform
```yaml
name: 'Terraform CI'

on:
  push:
    branches:
    - main

jobs:
  terraform:
    name: 'Terraform'
    runs-on: ubuntu-latest

    steps:
    - name: 'Checkout'
      uses: actions/checkout@v2

    - name: 'Terraform Init and Apply'
      run: |
        terraform init
        terraform apply -auto-approve
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```
This GitHub Actions workflow automates `terraform init` and `terraform apply` commands upon pushes to the main branch.

---

name: Lab-CICD-Integration
class: title
# Lab Exercise: Integrating Terraform with CI/CD
For this lab, you'll set up a basic CI/CD pipeline for a Terraform project using GitHub Actions (or your CI/CD tool of choice) to automate the deployment of infrastructure.
---

## Objective:
Create a CI/CD pipeline that automatically applies Terraform configurations upon changes in your VCS.
---

### Steps:
1. **Prepare Your Terraform Configuration:**
   Create a simple Terraform configuration that provisions an AWS S3 bucket.
2. **Set Up a GitHub Repository:**
   Initialize a GitHub repository and push your Terraform configuration.
3. **Configure GitHub Actions:**
   Create a `.github/workflows/terraform.yml` file in your repository with the content similar to the example provided.
4. **Configure Secrets:**
   Add your AWS credentials as secrets in the GitHub repository settings.
5. **Test Your Pipeline:**
   Make a change to your Terraform configuration, commit, and push the changes. Observe the automated pipeline run.
---

### Deliverable:
Submit the link to your GitHub repository containing the Terraform configuration and GitHub Actions workflow. Include a brief explanation of your pipeline and any observations or challenges encountered.

---


