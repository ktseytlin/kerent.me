---
layout: post
title:  "Intro to Terraform Modules"
date:   2016-04-22 09:00:00
categories: blog
---

# An Introduction to Terraform Modules

Goal: To provide information as to why Terraform modules are important and explain how to setup/run.

### What is Terraform?

Terraform is analogous to CloudFormation - it launches infrastructure. Based off my experience, I personally like Terraform more (and I find the capabilities of modules to be extremely powerful), but to each their own. Additionally, CloudFormation is exclusive to AWS whereas Terraform is compatible with multiple cloud providers.

### Why Terraform Modules?

Terraform Modules allow us to have cleaner code within our repository. Rather than maintaining all the Terraform scripts, we can just call the Terraform and pass the application variables.

At the end of the day, in AWS you are just constructing the same things over and over - EC2's, ELB's, ASG's, etc., and the only thing that differs between them is the configuration and variables. Instead, you can build out one module (for example, creating an EC2 instance) and then call the module and feed in those configuration variables.

Essentially, what we are building out is a skeleton that allows your to pass parameters through it, and then using those parameters it will go into the magical land of AWS and build you those resources. And better yet, you only need to build out those resources once. Then you just reuse it as many times as you want by pointing to that module.

Modules allow for **high reuseability** which means that you don't have to keep writing the same thing over and over. That sounds pretty awesome to me.

### Downloading Terraform

If you are working on a Mac, then you can use brew to download (if you don't have brew installed, I highly suggest you do so as it will make your life significantly easier).
```
brew install terraform
```

All you PC users, [check out this link for installation.] (https://www.terraform.io/intro/getting-started/install.html)

### Terraform Module Structure

Below are the files usually within a module:

* main.tf - This is the meat of your module. It is where you declare your module resources/providers/provisioners/etc.
* variables.tf - This outlines what variables are used in main.tf. However, you don't define the values of the variables here, although you can put in a default value. The variables here will be defined when the module is called.
* outputs.tf - Think of this as what you want the module to return to you. Could be an IP address, a name, etc.

I would recommend separating each module into it's own repository, rather than it's own folder with a repository. This will allow you to tag each module differently and make it easier for contribution to occur for each specific module.

In most resource types, there is a variable called "user_data". This variable is where you can pass through the location of bash scripts for the machine to execute. If you are working with Chef, recall that Capital One uses Chef bootstrap, all that is is a line of code that you can put in your bash script - you *do not need to create a Chef provisioner for your resources*. I have not testing the modules out yet with Ansible - more info on this later.

### Using a Module

Now that we have a basic handle on how things work, let's use a module and be super cool and build out machines - with minimal effort! This may be confusing at first but stick around for the ride because your life should get easier in the long term.

There are two ways to execute a module:

1. From the command line and passing in the parameters using `-var`
2. From another Terraform file that calls in the modules and specifies the parameters

Please note, that the two tactics below focus on using aws secret keys and access keys.


#### First Way: Calling the Module from the Command Line

If you're at work, you'll likely need to set up your proxy!

Now that you've set up the proxy, let's git clone a repository. For this example, let's git clone [this module] (https://github.kdc.capitalone.com/Athena/tfmodules-kafka).

```
git clone https://github.kdc.capitalone.com/Athena/tfmodules-kafka.git
git checkout master
git pull
```

In order to run the terraform commands, you need to run it from the directory that your module is in. First, we will use Terraform plan and then we will use Terraform apply.

We use Terraform plan to test run the module and see what will be created. This is incredibly useful. Plus if there are errors in your modules you will get error messages here, and then you will be able to resolve them. In the tfmodules-kafka that we cloned, I have put default values for everything except the aws access and secret keys -- never hard code these values are defaults, you never want to hardcode your passwords.
```
terraform plan -var "aws_access_key=MYACCESSKEY" -var "aws_secret_key=MYSECRETKEY"
```

Once our terraform plan action runs clean with no errors, let's run terraform apply. It is possible that you could run into additional errors here based on our Capital One AWS configurations. One example of this would be if you selected a certain instance type that doesn't allow for optimized EBS volumes. This error wouldn't show up in terraform plan, but would show up in terraform apply.
```
terraform apply -var "aws_access_key=MYACCESSKEY" -var "aws_secret_key=MYSECRETKEY"
```

Tired of all these machines you created? To delete you can either go into the AWS Website and delete the components, or from the command line you can use `terraform destroy`.

###### Calling Modules from an EC2 Instance To Create More Instances/Infrastructure

This section is useful if your team has their own Jenkins build machine in AWS.

What you would do in this circumstance is `git clone` the repository with the terraform module. Then run the command `terraform get`. After that you can use the same `terraform plan` and `terraform apply` commands as above.


#### Second Way: Calling the Module in Another File

Note: I am still testing out how to build the modules through Jenkins, so this section is subject to change.

You can also call Terraform Modules from different files. This is useful if you want to create an automated pipeline where modules are built our using Jenkins. In your Jenkins job, you would run a Terraform executing action in the Jenkins job and point it to the file that contains the modules.


Here is an example of creating a Terraform [configuration file that calls modules.](TODO:REPLACE URL) We created a file called athena.tf (but really, you can call it anything) and then called the module below:

```
module "kafka" {
  source = "git::https://github.com/...."
  aws_access_key = "${var.aws_access_key}"
  aws_secret_key = "${var.aws_secret_key}"
  aws_region = "${var.kafka_aws_region}"
  instance_type = "${var.kafka_instance_type}"
  ami_id = "${var.kafka_ami_id}"
  subnet_id = "${var.kafka_subnet_id}"
  user_data = "${var.kafka_user_data}"
  branch_name = "${var.branch_name}"
  env = "${var.env}"
  application_asv = "${var.application_asv}"
  CMDBEnvironment = "${var.CMDBEnvironment}"
  ebs_optimized = "${var.kafka_ebs_optimized}"
  component = "${var.kafka_component}"
  aws_key_name = "${var.aws_key_name}"
  asg_desired_size = "${var.kafka_asg_desired_size}"
  asg_min_size = "${var.kafka_asg_min_size}"
  asg_max_size = "${var.kafka_asg_max_size}"
}
```

The most important variables in the module is `source` - this gives the module the address of where to look.

Additionally, in declaring your resource you can specify how many times you want something to be replicated with the `count` variable. For example, I could specify the `count` in the resource to be 3. But I could also make that into a variable that is controlled by the calling of the module and that could create 3 instances within an ASG.

Even more importantly, remember that you can call the same module over and over again in the same file and pass different parameters to it. Just give them different names to save yourself a headache.

### Creating a Module

Now that you understand how modules work, you can totally make your own module! Keep in mind that it is not always necessary though to create your own modules - you can just borrow the work of others (using the source variable) and specify different configuration variables.

Follow the structure outlined above and scour the documentation to accurately create Terraform resources.

Have fun building!

### Things to Keep in Mind

* Due to the restrictiveness of VPN, if you are working on Terraform Modules over VPN you will likely encounter issues (even if you've set up your proxy). This might not actually be because of you, but rather because of VPN.
* Because of the way our Capital One AWS configuration is set up, you should not be talking to the outside world. Therefore, if there is a module you really like that you've found on the interwebs, bring it internally and then connect to it from our Github. Additionally, this is also why community groups of modules internally exist - contribute! :)
* **Do not nest modules.** It seems like an awesome idea and really cool, however, this is bad practice. You will likely end up breaking things.
* Can create graphical ways to view the module interaction. Check out Terraform documentation for more info on this.

### Good Links to Read:
* https://www.terraform.io/docs/modules/usage.html
* https://pulse.kdc.capitalone.com/docs/DOC-125487
* http://blog.lusis.org/blog/2015/10/12/terraform-modules-for-fun-and-profit/
* https://github.com/terraform-community-modules/tf_aws_ec2_instance
* https://github.com/terraform-community-modules/tf_aws_consul_cluster
