# Terraform learning notes

This information is baded on the books I bought... but not completely read :P

- [Learn Terraform the Hard way](https://leanpub.com/learnterraformthehardway) 
- [Terraform book](https://terraformbook.com/)

## Basic commands

* `terraform init`: Initializes and download all the plugins need in all *.tf files
* `terraform plan`: Executes a dry-run of what `apply` will execute
* `terraform apply`: Will execute the steps planned to have infrastructure in the same state as the terraform configuration.
* `terraform destroy`: Will destroy all the infrastructure (nice if you don't want to pay anymore)

## Basic configuration - Resource

basic.md:

```
provider "aws" {
    access_key = "..."
    secret_key = "..."
    region = "us-east-1"
}
resource "aws_vpc" "main" {
    cidr_block = "10.0.0.0/16"
}
```

1. when executin `terraform init`, will download the aws terraform plugin.
2. when executing `terraform plan`, will do a refresh of the current state on aws and check all the steps that has to perform but not doing anything (great to be sure what is going to happen)
3. `terraform apply` will creact a new VPC in aws

## Terraform state (tfstate files)

What terraform does is sees what is your desired state (what your .tf files says) and what is right now the current state of the infrastructure and do the small amount of steps to get the desired state.

This brings the problem on how to tight what is on the infrastructure provider (aws) and what is desired. how you map that the `aws_instance` resource in your tf file is the `i-123879217` in aws?

If you created the aws_instance in the first place, terraform will save this id's with the name of your resources in a files named tfstate.

to see what is currently in your tfstate you can use `terraform show`

If the instance was already created but you want to managed from now in terraform you should use the `terraform import` command



## variables and interpolation 

We can use other blocks information to fill current block, for example we can use variable blocks

```
variable "region" {
    description = "AWS region"
    default = "eu-west-1"
}

provider "aws" {
    access_key = ""
    secret_key = ""
    region = "${env.region}"
}
```

interpolation is always `${TYPE.NAME.ATTR}`, there are function to use in interpolation like, to look for the attibutes use `terraform show` after maing a apply:

* `file`: loads a local file into a variable for example:
```
resource  "aws_instance" "web" {
    ...
    userdata = "$(file("startup.sh"))""
}
```
* `lookup`: searches a key in a map

## Data

We not always want to create new things, sometimes we need to query the platform (aws) for some data

```
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

resource "aws_instance" "ubuntu" {
  ami = "${data.aws_ami.ubuntu.id}"
  instance_type = "t2.micro"
}
```


## Output

After provisioning the infrastructure maybe we need some infomation, for that we can output to our `.tf` files

```

resource "aws_instance" "ubuntu" {
  ami = "ami-456b493a"
  instance_type = "t2.micro"
}

output "ip" {
  value = "${aws_instance.ubuntu.associate_public_ip_address}"
}
```

`terraform output`: Shows all the last outputs in the tfstate

`terraform output ip`: will return the the value, great for doing `$(terraform output name)` on a bash script and load the value

## Dependencies

While using the `aws` provider/plugin the dependencies are handled by the provided,for example a subnet depends on a vpc, if you try to destroy the vpc it will automatically destroy the subnet.

But you can define your own dependencies, for example a web and a database ec2 instance.

```
resource "aws_instance" "web" {
  ami           = "ami-2757f631"
  instance_type = "t2.micro"

  depends_on = ["aws_instance.database"]
}
```

## Other commands

* `terraform refresh`: Refreshes the tfstate file with the information in aws

* `terraform import aws_vpc.main vpc-12a5a768`: Will load the data of the `vpc-12a5a768` created outside of the terraform into the tfstate so then we can track the changes

* `terraform taint "resource_name"`: will make this resource as tainted, which mean next time you do an apply the resource will be destroyed and created again

* `terraform graph | dot -Tpng > graph.png`: creates a `graph.png` image with the dependency graph of terraform

* `terraform console`: opens a REPL console
