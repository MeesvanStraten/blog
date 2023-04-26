---
author: Mees van Straten
datetime: 2023-04-26 12:00
title: Datadog dashboards with Terraform
slug: datadog-dashboards-with-terraform
featured: true
draft: false
tags:
  - datadog
  - terraform
  - monitoring
  - Monitoring-as-Code
ogImage: ""
description:
  Creating Datadog dashboards with Terraform.
---
This post originally appeared on [cloudlegends.nl](https://cloudlegends.nl/en/datadog-dashboards-with-terraform?latest)
<link rel="canonical" href="https://cloudlegends.nl/en/datadog-dashboards-with-terraform" />

# Introduction
In this blog post, we will look into how we can create Datadog dashboards with a MaC approach. Yes, I do love some Monitoring-as-Code every now and then. 
If you would like to see the Terraform code right away, [here you go!](https://github.com/MeesvanStraten/Datadog-Terraform-blog) We will use AWS as our platform to monitor, but you can use any other platform.

Managing our monitoring configuration with code has several advantages. By managing our monitoring as code we can easily create new environments or accounts and apply the existing configuration. We also get the ability to use version control and have our monitoring config centrally managed, this is great for collaboration and gives the possibility to enforce quality control like pull-requests and reviews.


# Prerequisites
- AWS account
- Datadog account, free plan is sufficient for this demo.
- Resources to monitor, we will use Lambda’s.


# Datadog
Datadog is a SaaS solution to observability. Datadog allows for collecting, visualizing and analyzing data from various sources such as servers, databases, applications and cloud based services. Datadog offers integrations with many popular technologies such as Docker and Kubernetes on different platforms like AWS and Azure, but it can be used in any environment.


# Terraform
Terraform is HashiCorp’s open-source solution to IaC(Infrastructure-as-code).
With Terraform, you declaratively manage your infrastructure in HCL (Hashicorp Config Language). With HCL as the declarative language, it means that you only declare the desired state, rather than the steps to get there.

Creating Infrastructure as Code has several advantages that we can also leverage for observability with Datadog. For starters,  we have the configuration as code instead of locking it in the Datadog UI. By managing our configuration as code we can easily replicate this to other Datadog accounts. Also, we can use version control to manage our Datadog configuration, and even deploy from a CI/CD pipeline.


# Let’s get started
Step-by-step, we will create a dashboard in Datadog and all the necessary configuration. We will start by creating a file called `provider.tf` , here we define the Datadog provider.

```
terraform {
 required_providers {
   datadog = {
     source = "DataDog/datadog"
   }
 }
}


provider "datadog" {
 api_key = var.TF_VAR_datadog_api_key
 app_key = var.TF_VAR_datadog_app_key
 api_url = var.datadog_api_url
}

```


Next up, we will create a file for declaring variables, let's call it `variables.tf`.
```
variable "TF_VAR_datadog_api_key" {
 type      = string
 sensitive = true
}


variable "TF_VAR_datadog_app_key" {
 type      = string
 sensitive = true
}


variable "datadog_api_url" {
 type = string
}
```


Now we can use these variables, as an example `var.datadog_api_url`.  Make sure to prefix with `var`. To store our variables, we will create a file named `terraform.tfvars`.
```
datadog_api_url = "https://api.datadoghq.eu/"
```

Heads Up! Never store confidential information like API keys / secrets in plaintext or add them to versioning systems. You can use tools like [Mozilla SOPS](https://github.com/mozilla/sops) or [HashiCorp Vault](https://www.vaultproject.io/) to securely store these items.


Next up, we will create two keys, the `API` key and the `APP` key.

## Create Datadog API key
- Navigate to `Organization settings`, then click the `API keys` tab.
- Click the `New Key` button,
- Enter a name for your key.
- Click `Create API key`.


## Create Datadog App key
- Navigate to `Organization settings`, then click the APP keys tab.
- Click the `New Key` button,
- Enter a name for your key.
- Click `Create APP key`.

Add these keys to your environment variables with the `TF_VAR prefix`, Terraform will automatically recognize the variables and assign them to the earlier declared variables and use them for the Datadog provider block.

```
export TF_VAR_vcs_datadog_app_key=yourAppKey
export TF_VAR_vcs_datadog_api_key=yourAPIKey
```


# AWS Integration
At the time of writing the AWS integration can not be configured with Terraform, this is a one time thing for each AWS account. To create the integration, perform the following steps:

Click Integrations in the sidebar.
Click create new key and select the region you want to monitor.
Select the API key we created earlier.
Leave the default options and click create CloudFormation.
This will take you to CloudFormation in the AWS Console, review the stack (defaults should be good) and click Create Stack.

This can take a few minutes, after some time you should have a properly configured AWS integration with Datadog.


# Create the dashboard
Now it is time to create a dashboard, let's start by creating a file called `dashboard.tf`.

We will add two group widgets, which we can use to organise and group our widgets within them. Within the Lambda group we will create three widgets that display total invocations, invocations per function and the potential errors in functions.

In the Infrastructure group, we will create a host map that displays all our hosts and group them by region.

```
resource "datadog_dashboard" "ordered_dashboard" {
 title       = "Our Dashboard with Terraform"
 layout_type = "ordered"

 widget {
   group_definition {
     title            = "Lambda"
     layout_type      = "ordered"
     background_color = "green"


     widget {
       query_value_definition {
         live_span = "1mo"
         title     = "Lambda Invocations"
         precision = 0
         autoscale = true
         request {
           q = "sum:aws.lambda.invocations{*}.as_count()"
           conditional_formats {
             comparator = "<"
             value      = "2500"
             palette    = "white_on_green"
           }
         }
       }
     }
     widget {
       toplist_definition {
         title     = "Top 10 Lambda Invocations"
         live_span = "1mo"
         request {
           style {
             palette = "dog_classic"
           }
           q = "sum:aws.lambda.invocations{*} by {functionname}.as_count()"
         }
       }
     }
     widget {


       query_value_definition {
         title     = "Lambda Errors"
         live_span = "1d"
         autoscale = true
         request {
           q = "sum:aws.lambda.errors{*}.as_count()"
           conditional_formats {
             comparator = "<"
             value      = "5"
             palette    = "white_on_green"
           }
           conditional_formats {
             comparator = ">"
             value      = "5"
             palette    = "white_on_red"
           }
         }
       }
     }
   }
 }
 widget {
   group_definition {
     title            = "Infrastructure"
     layout_type      = "ordered"
     background_color = "orange"

     widget {
       hostmap_definition {
         title           = "Host Map"
         group           = ["region"]
         no_group_hosts  = true
         no_metric_hosts = true
         request {
           fill {
             q = "avg:system.cpu.user{*} by {host}"
           }
         }
       }
     }
   }
 }
}
```

# Apply the Terraform
Now it is time to see what we created, run `Terraform plan` and review the changes. You should have a single resource to add, that's our dashboard.

Now run `Terraform apply`, still 1 to add and type in yes.
We now have our Terraformed dashboard live!

You can visit your dashboard in the sidebar by navigating to the dashboard list. Your dashboard should look similar if you have some Lambda’s running.

If you would like to clean up all resources you can run `terraform destroy`.


# Conclusion
We experienced how we can use Terraform for creating Datadog dashboards and highlighted what the advantages can be. Hopefully in the near future we can also create the AWS integration with Terraform. Furthermore the Terraform resources are quite extensive and almost all functionality available in the Datadog UI can be used with Terraform as well. You can find the Datadog provider [here](https://registry.terraform.io/providers/DataDog/datadog/latest/docs).

Thank you for reading this article about Datadog and Terraform! Feel free to [checkout the repository](https://github.com/MeesvanStraten/Datadog-Terraform-blog) containing the Terraform code.
