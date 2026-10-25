# terraform-scaleway-serverless_container

Terraform module to manage [Scaleway Serverless Containers](https://www.scaleway.com/en/serverless-containers/).

## Features

- Deploy a Serverless Container from any OCI-compliant registry image
- Optional custom DNS domain with automatic CNAME record creation
- Automatic IAM API key creation and storage in Scaleway Secret Manager for private containers
- VPC private network attachment support

## Usage

### Public container

```hcl
module "container" {
  source  = "scaleway-terraform-modules/serverless-container/scaleway"
  version = "~> 0.0"

  name                   = "my-service"
  container_namespace_id = scaleway_container_namespace.main.id

  image_name  = "rg.fr-par.scw.cloud/my-namespace/my-service"
  app_version = "1.0.0"

  privacy  = "public"
  protocol = "http1"
  port     = 8080

  min_scale = 0
  max_scale = 5

  environment_variables = {
    LOG_LEVEL = "info"
  }
}
```

### Private container with IAM authentication

For private containers, pass the ID of an existing IAM application. The module will create an API key for that application and store the secret key in Scaleway Secret Manager.

```hcl
data "scaleway_iam_application" "my_app" {
  name = "my-iam-application"
}

module "container" {
  source  = "scaleway-terraform-modules/serverless-container/scaleway"
  version = "~> 0.0"

  name                   = "my-service"
  container_namespace_id = scaleway_container_namespace.main.id
  project_id             = var.project_id

  image_name  = "rg.fr-par.scw.cloud/my-namespace/my-service"
  app_version = "1.0.0"

  privacy = "private"
  port    = 8080

  iam_application_id = data.scaleway_iam_application.my_app.application_id

  secret_path = "/my-team/my-service"
}
```

To authenticate against a private container, pass the stored secret key as the `X-Auth-Token` HTTP header.

### With a custom DNS domain

```hcl
module "container" {
  source  = "scaleway-terraform-modules/serverless-container/scaleway"
  version = "~> 0.0"

  name                   = "my-service"
  container_namespace_id = scaleway_container_namespace.main.id
  project_id             = var.project_id

  image_name  = "rg.fr-par.scw.cloud/my-namespace/my-service"
  app_version = "1.0.0"

  privacy = "public"
  port    = 8080

  dns_zone       = "example.com"
  record         = "my-service"
  dns_project_id = var.dns_project_id
}
```

## Examples

- [Basic](./examples/basic) — minimal deployment of a public container


<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement_terraform) | >= 1.11 |
| <a name="requirement_scaleway"></a> [scaleway](#requirement_scaleway) | >= 2.60.0 |

## Resources

| Name | Type |
|------|------|
| [scaleway_container.main](https://registry.terraform.io/providers/scaleway/scaleway/latest/docs/resources/container) | resource |
| [scaleway_container_domain.main](https://registry.terraform.io/providers/scaleway/scaleway/latest/docs/resources/container_domain) | resource |
| [scaleway_domain_record.main](https://registry.terraform.io/providers/scaleway/scaleway/latest/docs/resources/domain_record) | resource |
| [scaleway_iam_api_key.main](https://registry.terraform.io/providers/scaleway/scaleway/latest/docs/resources/iam_api_key) | resource |
| [scaleway_secret.container_secret_key](https://registry.terraform.io/providers/scaleway/scaleway/latest/docs/resources/secret) | resource |
| [scaleway_secret_version.container_secret_key](https://registry.terraform.io/providers/scaleway/scaleway/latest/docs/resources/secret_version) | resource |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_app_version"></a> [app_version](#input_app_version) | Container image version/tag | `string` | n/a | yes |
| <a name="input_container_namespace_id"></a> [container_namespace_id](#input_container_namespace_id) | ID of the container namespace where to deploy the container | `string` | n/a | yes |
| <a name="input_image_name"></a> [image_name](#input_image_name) | Container image name without tag (e.g., registry.example.com/image) | `string` | n/a | yes |
| <a name="input_name"></a> [name](#input_name) | Name of the serverless container | `string` | n/a | yes |
| <a name="input_port"></a> [port](#input_port) | Port on which the container listens | `number` | n/a | yes |
| <a name="input_cpu_limit"></a> [cpu_limit](#input_cpu_limit) | Maximum vCPU allocated to the container (in millicores, e.g. 1000 = 1 vCPU) | `number` | `null` | no |
| <a name="input_deploy"></a> [deploy](#input_deploy) | Whether to deploy the container | `bool` | `true` | no |
| <a name="input_dns_project_id"></a> [dns_project_id](#input_dns_project_id) | Scaleway project ID where the DNS zones and records are managed (required when dns_zone is set) | `string` | `null` | no |
| <a name="input_dns_zone"></a> [dns_zone](#input_dns_zone) | DNS zone for the container custom domain (optional — no DNS resources are created if not set) | `string` | `null` | no |
| <a name="input_environment_variables"></a> [environment_variables](#input_environment_variables) | Plain-text environment variables to inject into the container | `map(string)` | `{}` | no |
| <a name="input_iam_application_id"></a> [iam_application_id](#input_iam_application_id) | ID of the IAM application for which an API key will be created to authenticate against the private container. Only used when privacy is 'private'. | `string` | `null` | no |
| <a name="input_max_scale"></a> [max_scale](#input_max_scale) | Maximum number of container instances to scale to | `number` | `5` | no |
| <a name="input_memory_limit"></a> [memory_limit](#input_memory_limit) | Maximum memory allocated to the container (in MB) | `number` | `null` | no |
| <a name="input_min_scale"></a> [min_scale](#input_min_scale) | Minimum number of container instances to maintain | `number` | `1` | no |
| <a name="input_privacy"></a> [privacy](#input_privacy) | Privacy policy for the container (public or private) | `string` | `"private"` | no |
| <a name="input_private_network_id"></a> [private_network_id](#input_private_network_id) | VPC private network ID to run the container in | `string` | `null` | no |
| <a name="input_project_id"></a> [project_id](#input_project_id) | Scaleway project ID where the serverless container will be created | `string` | `null` | no |
| <a name="input_protocol"></a> [protocol](#input_protocol) | Protocol to use for the container (http1 or h2c) | `string` | `"http1"` | no |
| <a name="input_record"></a> [record](#input_record) | DNS record name for the container, without the zone suffix (optional — no DNS resources are created if not set) | `string` | `null` | no |
| <a name="input_secret_environment_variables"></a> [secret_environment_variables](#input_secret_environment_variables) | Secret environment variables to inject into the container | `map(string)` | `{}` | no |
| <a name="input_secret_name"></a> [secret_name](#input_secret_name) | Name of the Scaleway Secret used to store the IAM API key. Defaults to '<NAME>-AUTH-SECRET-KEY'. | `string` | `null` | no |
| <a name="input_secret_path"></a> [secret_path](#input_secret_path) | Path of the Scaleway Secret used to store the IAM API key. Defaults to '/containers'. | `string` | `null` | no |
| <a name="input_tags"></a> [tags](#input_tags) | List of tags to apply to the serverless container resources | `list(string)` | `[]` | no |
| <a name="input_timeout"></a> [timeout](#input_timeout) | Maximum time in seconds the container can run before timing out | `number` | `300` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_container_endpoint"></a> [container_endpoint](#output_container_endpoint) | The default endpoint URL of the serverless container |
| <a name="output_container_id"></a> [container_id](#output_container_id) | The ID of the serverless container |
| <a name="output_container_name"></a> [container_name](#output_container_name) | The name of the serverless container |
| <a name="output_container_status"></a> [container_status](#output_container_status) | The status of the serverless container |
| <a name="output_dns_record_fqdn"></a> [dns_record_fqdn](#output_dns_record_fqdn) | The fully qualified domain name of the DNS record (only set when DNS is configured) |
| <a name="output_domain_hostname"></a> [domain_hostname](#output_domain_hostname) | The custom domain hostname for the container (only set when DNS is configured) |
| <a name="output_domain_url"></a> [domain_url](#output_domain_url) | The full URL of the container with custom domain (only set when DNS is configured) |
| <a name="output_iam_api_key_secret_id"></a> [iam_api_key_secret_id](#output_iam_api_key_secret_id) | The ID of the secret storing the IAM API key (only set for private containers with iam_application_id) |
| <a name="output_iam_api_key_secret_name"></a> [iam_api_key_secret_name](#output_iam_api_key_secret_name) | The name of the secret storing the IAM API key (only set for private containers with iam_application_id) |
<!-- END_TF_DOCS -->

## Authors

Module is maintained with help from [the community](https://github.com/scaleway-terraform-modules/terraform-scaleway-serverless-container/graphs/contributors).

## License

Mozilla Public License 2.0 Licensed. See [LICENSE](https://github.com/scaleway-terraform-modules/terraform-scaleway-bucket/tree/master/LICENSE) for full details.