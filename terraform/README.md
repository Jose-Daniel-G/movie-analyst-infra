<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
| ---- | ------- |
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.10 |
| <a name="requirement_aws"></a> [aws](#requirement\_aws) | >= 5.80, < 7.0 |
| <a name="requirement_random"></a> [random](#requirement\_random) | ~> 3.6 |

## Providers

| Name | Version |
| ---- | ------- |
| <a name="provider_aws"></a> [aws](#provider\_aws) | 6.73.0 |
| <a name="provider_random"></a> [random](#provider\_random) | 3.9.1 |

## Modules

| Name | Source | Version |
| ---- | ------ | ------- |
| <a name="module_alb_internal"></a> [alb\_internal](#module\_alb\_internal) | ./modules/alb | n/a |
| <a name="module_alb_public"></a> [alb\_public](#module\_alb\_public) | ./modules/alb | n/a |
| <a name="module_api"></a> [api](#module\_api) | ./modules/instance | n/a |
| <a name="module_bastion"></a> [bastion](#module\_bastion) | ./modules/instance | n/a |
| <a name="module_iam"></a> [iam](#module\_iam) | ./modules/iam | n/a |
| <a name="module_monitoring"></a> [monitoring](#module\_monitoring) | ./modules/monitoring | n/a |
| <a name="module_network"></a> [network](#module\_network) | ./modules/network | n/a |
| <a name="module_rds"></a> [rds](#module\_rds) | ./modules/rds | n/a |
| <a name="module_security"></a> [security](#module\_security) | ./modules/security | n/a |
| <a name="module_ui"></a> [ui](#module\_ui) | ./modules/instance | n/a |

## Resources

| Name | Type |
| ---- | ---- |
| [aws_key_pair.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/key_pair) | resource |
| [aws_ssm_parameter.this](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ssm_parameter) | resource |
| [random_password.db_admin](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password) | resource |
| [random_password.db_app](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password) | resource |
| [aws_availability_zones.available](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/availability_zones) | data source |
| [aws_ssm_parameter.al2023](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/ssm_parameter) | data source |

## Inputs

| Name | Description | Type | Default | Required |
| ---- | ----------- | ---- | ------- | :------: |
| <a name="input_alert_email"></a> [alert\_email](#input\_alert\_email) | Email that receives CloudWatch alarm notifications (you must confirm the SNS subscription). | `string` | n/a | yes |
| <a name="input_allowed_ssh_cidr"></a> [allowed\_ssh\_cidr](#input\_allowed\_ssh\_cidr) | Your public IP in CIDR form, e.g. 190.1.2.3/32. Only this range can SSH to the bastion. | `string` | n/a | yes |
| <a name="input_db_engine_version"></a> [db\_engine\_version](#input\_db\_engine\_version) | n/a | `string` | `"8.4"` | no |
| <a name="input_db_name"></a> [db\_name](#input\_db\_name) | n/a | `string` | `"movie_db"` | no |
| <a name="input_public_key_path"></a> [public\_key\_path](#input\_public\_key\_path) | Path to the SSH public key installed on all instances. | `string` | `"~/.ssh/id_ed25519.pub"` | no |
| <a name="input_region"></a> [region](#input\_region) | n/a | `string` | `"us-east-2"` | no |

## Outputs

| Name | Description |
| ---- | ----------- |
| <a name="output_app_url"></a> [app\_url](#output\_app\_url) | Public URL of the application. |
| <a name="output_bastion_public_ip"></a> [bastion\_public\_ip](#output\_bastion\_public\_ip) | n/a |
| <a name="output_dashboard_url"></a> [dashboard\_url](#output\_dashboard\_url) | n/a |
| <a name="output_internal_alb_dns"></a> [internal\_alb\_dns](#output\_internal\_alb\_dns) | n/a |
| <a name="output_rds_endpoint"></a> [rds\_endpoint](#output\_rds\_endpoint) | n/a |
| <a name="output_ssh_to_bastion"></a> [ssh\_to\_bastion](#output\_ssh\_to\_bastion) | Use -A so your key is forwarded and Ansible can reach the private instances. |
<!-- END_TF_DOCS -->