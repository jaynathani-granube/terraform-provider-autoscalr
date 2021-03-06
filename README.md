AutoScalr Terraform Provider
=========================

- Website: https://www.autoscalr.com

Requirements
------------

-	[Terraform](https://www.terraform.io/downloads.html) 0.10.x
-	[Go](https://golang.org/doc/install) 1.9 (to build the provider plugin)

Downloading The Provider
---------------------

Clone repository to: `$GOPATH/src/github.com/autoscalr/terraform-provider-autoscalr` by either:

```sh
$ go get github.com/autoscalr/terraform-provider-autoscalr
```

or

```sh
$ git clone git@github.com:autoscalr/terraform-provider-autoscalr $GOPATH/src/github.com/autoscalr/terraform-provider-autoscalr
```

Building The Provider
---------------------

Enter the provider directory and run the build script to build the provider for your architecture

```sh
$ cd $GOPATH/src/github.com/autoscalr/terraform-provider-autoscalr
$ ./build.sh
```

Copy the resulting terraform-provider-autoscalr file in that directory to the terraform plugins directory
for your terraform workspace located in

```sh
$TERRAFORM_WKSP/terraform.d/plugins/{ARCH}/
```


Using the provider
----------------------

The AutoScalr provider requires that you specify the api_key associated with your AutoScalr account.
It is recommended to specify it via the environment variable AUTOSCALR_API_KEY.
Alternatively you can specify it as a parameter when initializing the provider in your .tf file as:

```sh
provider "autoscalr" {
  api_key = "your API key value"
}
```

If you do not know what your API key value is for your AutoScalr account contact, support@autoscalr.com.

If you do not have an AutoScalr account, you can sign up for free trial at: http://autoscalr.com/get-started/

The exampleUse.tf file shows how to use the autoscalr provider to extend an AWS autoscaling group:

```sh
provider "aws" {
  region = "us-east-1"
}
provider "autoscalr" {
  // You either need to specify the api_key here or via the AUTOSCALR_API_KEY enviroment variable
  //api_key = "yourKey"
}

resource "aws_launch_configuration" "test_lc" {
  name_prefix   = "test-lc-"
  image_id      = "ami-8c1be5f6"  // Base Amazon Linux AMI in us-east-1
  instance_type = "t1.micro"

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_autoscaling_group" "myAppASG" {
  name_prefix                 = "test-asg-"
  availability_zones          = ["us-east-1a", "us-east-1b","us-east-1c"]
  max_size                    = 2
  min_size                    = 0
  desired_capacity            = 0
  health_check_grace_period   = 300
  health_check_type           = "EC2"
  force_delete                = true
  launch_configuration        = "${aws_launch_configuration.test_lc.name}"
  lifecycle {
    create_before_destroy     = true
  }
  suspended_processes         = ["AZRebalance"]  // Recommended to keep ASG from fighting AutoScalr AZ Rebalancing
}

resource "autoscalr_autoscaling_group" "asr4myAppASG" {
  aws_region                  = "us-east-1"
  aws_autoscaling_group_name  = "${aws_autoscaling_group.myAppASG.name}"
  instance_types              = ["c3.large","c3.xlarge"]
  display_name                = "myFirstAutoScalrApp"
  max_spot_percent_total      = 85
  max_spot_percent_one_market = 25
}
```

If you copy this file to your Terraform workspace that has the plugin installed and set your AUTOSCALR_API_KEY via environment
variable or parameter, you should be able to build the simple example stack by:

 ```sh
 $ terraform init
 $ terraform plan
 $ terraform apply
 ```

All the attributes available on the resource are documented [here](https://github.com/autoscalr/terraform-provider-autoscalr/blob/master/website/docs/autoscalr_autoscaling_group.md) (website/docs/)

