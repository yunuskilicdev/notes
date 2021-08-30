**confd** is a lightweight configuration management tool focused on:

keeping local configuration files up-to-date using data stored in etcd, consul, dynamodb, redis, vault, zookeeper, aws ssm parameter store or env vars and processing template resources.
reloading applications to pick up new config file changes[0].

**AWS SSM**

AWS Systems Manager allows you to centralize operational data from multiple AWS services and automate tasks across your AWS resources. You can create logical groups of resources such as applications, different layers of an application stack, or production versus development environments. With Systems Manager, you can select a resource group and view its recent API activity, resource configuration changes, related notifications, operational alerts, software inventory, and patch compliance status. You can also take action on each resource group depending on your operational needs. Systems Manager provides a central place to view and manage your AWS resources, so you can have complete visibility and control over your operations.[1]

Confd is related with configuration management. That's why I am interested in SSM parameter store.

**SSM Parameter Store**

AWS Systems Manager provides a centralized store to manage your configuration data, whether plain-text data such as database strings or secrets such as passwords. This allows you to separate your secrets and configuration data from your code. Parameters can be tagged and organized into hierarchies, helping you manage parameters more easily. For example, you can use the same parameter name, "db-string", with a different hierarchical path, "dev/db-string” or “prod/db-string", to store different values. Systems Manager is integrated with AWS Key Management Service (KMS), allowing you to automatically encrypt the data you store. You can also control user and resource access to parameters using AWS Identity and Access Management (IAM). Parameters can be referenced through other AWS services, such as Amazon Elastic Container Service, AWS Lambda, and AWS CloudFormation.[1]

One such service is SSM Parameter Store which is a secured and managed key/value store perfect for storing parameters, secrets, and configuration information. However, in April of 2018, AWS also introduced another service called AWS Secrets Manager that offers similar functionality. Given that both services kind of do the same thing, which to choose isn’t clear. With that in mind, let us take a look at the similarities and differences of these two services to better understand which service will best fit your architectural needs.[2]

SSM Parameter Store is free, that's why I prefer it instead of AWS Secrets Manager.

````
aws ssm put-parameter --name "/myapp/database/url" --type "String" --value "db.example.com"
aws ssm put-parameter --name "/myapp/database/user" --type "SecureString" --value "rob"
````

Above commands put two parameter to SSM Parameter Store

The confdir is where template resource configs and source templates are stored.

````
sudo mkdir -p /etc/confd/{conf.d,templates}
````

Template resources are defined in [TOML](https://github.com/toml-lang/toml) config files under the confdir.

/etc/confd/conf.d/myconfig.toml

````
[template]
src = "myconfig.conf.tmpl"
dest = "/tmp/myconfig.conf"
keys = [
    "/myapp/database/url",
    "/myapp/database/user",
]
````

Source templates are [Golang text templates](https://pkg.go.dev/text/template#pkg-overview).

/etc/confd/templates/myconfig.conf.tmpl

````
[myconfig]
database_url = {{getv "/myapp/database/url"}}
database_user = {{getv "/myapp/database/user"}}
````

````
confd -onetime -backend ssm
````

````
$ cat /tmp/myconfig.conf 
[myconfig]
database_url = db.example.com
database_user = rob

````

0-) https://github.com/kelseyhightower/confd

1-) https://aws.amazon.com/systems-manager/features/

2-) https://www.1strategy.com/blog/2019/02/28/aws-parameter-store-vs-aws-secrets-manager/