# AWS Terraform Best Practices

The purpose of this repository is to provide Terraform best practices as well as agreed-upon ways to work effectively as a team that will help you deliver high quality Terraform code.

<!-- TOC -->

- [AWS Terraform Best Practices](#aws-terraform-best-practices)
  - [Contributing](#contributing)
  - [General Guidelines](#general-guidelines)
  - [Terraform Guidelines](#terraform-guidelines)
    - [Terraform Version Management](#terraform-version-management)
    - [Standard module structure](#standard-module-structure)
    - [Reusable Module structure](#reusable-module-structure)
      - [Module version pinning using Git commit hash](#module-version-pinning-using-git-commit-hash)
    - [Naming convention](#naming-convention)
    - [Stateful Resources](#stateful-resources)
    - [Built-In Formatting and Validation](#built-in-formatting-and-validation)
    - [Expressions Complexity](#expressions-complexity)
    - [Conditional Values](#conditional-values)
    - [Iterated Resources](#iterated-resources)
    - [Attachment Resources](#attachment-resources)
    - [Custom Objects](#custom-objects)
    - [Workspaces](#workspaces)
    - [Remote state](#remote-state)
    - [Default Tags](#default-tags)
    - [Plan](#plan)
    - [Secrets](#secrets)
    - [Pre-Commit Hooks](#pre-commit-hooks)
    - [Infrastructure testing](#infrastructure-testing)
  - [Terraform Module Creation](#terraform-module-creation)
    - [Source Repository](#source-repository)
    - [Terraform Registry](#terraform-registry)
    - [Module Sources](#module-sources)
      - [Registry](#registry)
        - [Hashicorp Terraform Registry](#hashicorp-terraform-registry)
        - [Terraform Cloud](#terraform-cloud)
        - [Terraform Enterprise](#terraform-enterprise)
      - [VCS Providers](#vcs-providers)
        - [GitHub HTTPS](#github-https)
        - [Generic Git Repository HTTPS](#generic-git-repository-https)
        - [Generic Git Repository SSH](#generic-git-repository-ssh)
  - [Terraform Backend Configuration](#terraform-backend-configuration)
    - [Terraform Cloud/Enterprise Configuration](#terraform-cloudenterprise-configuration)
    - [Amazon S3 Configuration](#amazon-s3-configuration)
  - [Security](#security)
  - [License Summary](#license-summary)

<!-- /TOC -->

## Contributing

It will be possible to contribute to this repository once it is published. Contribution requirements will be clarified shortly.

## General Guidelines

- Always break down tasks into smaller simpler components, which only have access to the information that they need. Doing too much in one chunk isn’t manageable or maintainable when creating, understanding, testing and bug fixing code. Always functionize code and inject dependencies/parameters. This also makes it clearer as to what objects need to function and, thus, makes the code easier to test.
- Keep the code [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). When you repeat lines of code, you could be copying a bug, a non-optimal approach, or an external dependency. Reviewing highly repetitive code is also not optimal and can be a waste of time for reviewer’s.
- Make sure your implementation has been minimally tested in order to verify that everything still works as it should. Automated tests can also help to identify issues early in the development process before they become a problem.
- Write code to make it easy to read and understand and use descriptive names so your colleagues can decipher everything. If a piece of code is to complex to understand using only descriptive naming, use comments to explain what it is doing and why. All of our code is written with the awareness that it will be peer-reviewed.
- If you are stuck working on your task for 30 minutes, raise it to the team. Ask for help if anyone is available to have a pair or mob programming session with you, depending on the complexity of the task
- Look for existing frameworks, review and contribute to them. If a public package/module already implement the pattern you want, why re-invent the wheel?
- Recognize when it's time to refactor. If a piece of code is straying from its original intent, and there's a better way of doing it you should flag it for refactoring so the team can analyze and discuss the problem.
- Commit to the source code repository often, at a minimum once a day. Regularly pushing up your code will help ensure you don’t lose any work if something goes wrong with your computer or software, and it also makes it easier to track changes and keep track of who made which changes and when.
- Contributing guidelines must be documented in the CONTRIBUTING.md file. This file must also include the tools setup required to make changes to the source code.
- Specifies files that Git should ignore in a .gitignore file at the root of the repository.
- Place any additional documentation in a docs/ subdirectory.
- Include a [CODEOWNERS](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners) file, documenting who is responsible for the module. Before any merge request is merged, an owner should approve it.
- The main branch is the primary development branch and represents the latest approved code. The main branch is [protected](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches).
- Development happens on feature and bug-fix branches that branch off of the main branch.
  - Name feature branches **feature/feature-name**.
  - Name bug-fix branches **fix/bugfix-name**.
- To prevent merge conflicts, always pull the latest version of the main branch into your feature/fix branch.

## Terraform Guidelines

### Terraform Version Management

Use [tfenv](https://github.com/tfutils/tfenv/) to install and switch between specific versions of Terraform.

### Standard module structure

The standard [module structure](https://developer.hashicorp.com/terraform/language/modules/develop#module-structure) expects the layout documented below. Terraform folders/files must exist in the root directory of the repository.

- **Root module/directory**: This should be the primary entrypoint for the module and is expected to be opinionated. More complex architectures will use specific nested modules to create lightweight abstractions, so that you can describe infrastructure in terms of its architecture, rather than directly in terms of physical objects.
- **README**: The root module and any nested modules should have README files. This file must be named **README.md**. It should contain a description of the module and what it should be used for. If you want to include an example for how this module can be used in combination with other resources, put it in an `examples` directory. Consider including a visual diagram depicting the infrastructure resources the module may create and their relationship.

  The README doesn't need to document inputs or outputs of the module because the terraform-docs pre-commit hook will automatically generate this. If you are linking to a file or embedding an image contained in the repository itself, use a relative url (e.g., `./CONTRIBUTE.md`) or a [commit-specific absolute URL](https://docs.github.com/en/repositories/working-with-files/using-files/getting-permanent-links-to-files) (e.g., `https://github.com/aws-samples/aws-terraform-template/blob/84930f010d7a60223df0ebd360b4df69319187e2/.pre-commit-config.yaml`) so the link won't point to the wrong version of a resource in the future.
  >  You can press the `Y` key when viewing a file to transform the URL to another URL that points to the particular commit.
- **main.tf**: This is the primary entrypoint. For a simple module, this may be where all the resources are created. For a complex module, resource creation may be split into multiple files but any nested module calls should be in the main file.
- **variables.tf** and **outputs.tf**: Contain the declarations for variables and outputs, respectively. All variables and outputs should have one or two sentence descriptions that explain their purpose. This is used for documentation. See the documentation for [variable configuration](https://developer.hashicorp.com/terraform/language/values/variables) and [output configuration](https://developer.hashicorp.com/terraform/language/values/outputs) for more details.
  - All variables must have a defined type.
  - The variable declaration can also include a default argument. If present, the variable is considered to be optional and the default value will be used if no value is set when calling the module or running Terraform. The default argument requires a literal value and cannot reference other objects in the configuration. To make a variable required for user to set, omit a default in the variable declaration and consider if setting `nullable = false` makes sense.
  - For variables that have environment-independent values (such as disk size), provide default values.
  - For variables that have environment-specific values (such as `project_id`), don't provide default values. This way, the calling module must provide meaningful values.
  - Use empty defaults for variables (like empty strings or lists) only when leaving the variable empty is a valid preference that the underlying APIs don't reject.
  - Be judicious in your use of variables. Only parameterize values that must vary for each instance or environment. When deciding whether to expose a variable, ensure that you have a concrete use case for changing that variable. If there's only a small chance that a variable might be needed, don't expose it.
    - Adding a variable with a default value is backwards-compatible.
    - Removing a variable is backwards-incompatible.
    - In cases where a literal is reused in multiple places, you can use a local value without exposing it as a variable.
  - Don't pass outputs directly through input variables, because doing so prevents them from being properly added to the dependency graph. To ensure that [implicit dependencies](https://learn.hashicorp.com/terraform/getting-started/dependencies.html) are created, make sure that outputs reference attributes from resources. Instead of referencing an input variable for an instance directly, pass the attribute.
- **locals.tf**: Contains local values that assign a name to an expression, so a name can be used multiple times within a module instead of repeating the expression. Local values are like a function's temporary local variables. The expressions in local values are not limited to literal constants; they can also reference other values in the module in order to transform or combine them, including variables, resource attributes, or other local values.
- **providers.tf**: Contains the [terraform block](https://www.terraform.io/language/settings) and [provider blocks](https://developer.hashicorp.com/terraform/language/providers/configuration#provider-configuration-1). `provider` blocks must only be declared in root modules by consumers of modules.

  If using Terraform Cloud/Enterprise, also add an empty [cloud block](https://developer.hashicorp.com/terraform/cli/cloud/settings#the-cloud-block). The `cloud` block is configured entirely through [environment variables](https://developer.hashicorp.com/terraform/cli/cloud/settings#environment-variables) and [environment variables credentials]( https://developer.hashicorp.com/terraform/cli/config/config-file#environment-variable-credentials) as part of a CICD Pipeline.
- **versions.tf**: Contains the [required_providers](https://developer.hashicorp.com/terraform/language/providers/requirements#requiring-providers) block. Each Terraform module must declare which providers it requires, so that Terraform can install and use them.
- **data.tf**: For simple configuration, put [data sources](https://developer.hashicorp.com/terraform/language/data-sources) next to the resources that reference them. For example, if you are fetching an image to be used in launching an instance, place it alongside the instance instead of collecting data resources in their own file. If the number of data sources becomes large, consider moving them to a dedicated `data.tf` file.
- **.tfvars files**: For root modules, provide variables by using a `.tfvars` variables file. For consistency, name variable files `terraform.tfvars`. Place common values at the root of the repository and environment specific values within the `envs/` folder.
- **Nested modules**: Nested modules must exist under the `modules/` subdirectory. Any nested module with a `README.md` is considered usable by an external user. If a README doesn't exist, it is considered for internal use only. Nested modules should be used to split complex behavior into multiple small modules that users can carefully pick and choose.

  If the root module includes calls to nested modules, they should use relative paths like ./modules/sample-module so that Terraform will consider them to be part of the same repository or package, rather than downloading them again separately.

  If a repository or package contains multiple nested modules, they should ideally be composable by the caller, rather than calling directly to each other and creating a deeply-nested tree of modules.
- **Examples**: Examples of using the module should exist under the `examples/` subdirectory at the root of the repository. Each example may have a README to explain the goal and usage of the example. Examples for submodules should also be placed in the root `examples/` directory.

  Because examples will often be copied into other repositories for customization, any module blocks should have their source set to the address an external caller would use, not to a relative path.
- **Service named files**: Often users want to create several files and separate terraform resources by service. This urge should be stifled as much as possible in favor of defining resources in `main.tf`. If a collection of resources, for example IAM Roles and Policies, exceed 150 lines then it is reasonable to break that into its own files such as `iam.tf`. Otherwise all resource code should be defined in the `main.tf`.
- **Custom Scripts**: Use scripts only when necessary. The state of resources created through scripts is not accounted for or managed by Terraform. Use them only when Terraform resources don't support the desired behavior. Put custom scripts called by Terraform in a `scripts/` directory.
- **Helper Scripts**: Organize helper scripts that aren't called by Terraform in a `helpers/` directory. Document helper scripts in the `README.md` file with explanations and example invocations. If helper scripts accept arguments, provide argument-checking and `--help` output.
- **Static Files**: Static files that Terraform references but doesn't execute (such as startup scripts loaded onto Amazon EC2 instances) must be organized into a `files/` directory. Place lengthy HereDocs in external files, separate from their HCL. Reference them with the [file() function](https://www.terraform.io/language/functions/file).
- **Templates**: For files that are read in by using the Terraform [templatefile function](https://www.terraform.io/docs/configuration/functions/templatefile.html), use the file extension `.tftpl`. Templates must be placed in a `templates/` directory.

### Reusable Module structure

> You can find some reusable modules examples [here](https://github.com/terraform-aws-modules).

This section is about creating re-usable modules that other configurations can include using `module` blocks. In principle any combination of resources and other constructs can be factored out into a module, but over-using modules can make your overall Terraform configuration harder to understand and maintain, so you must use them with moderation.

Re-usable modules that are defined using all of the same concepts we use in root modules. To define a module, create a new directory for it and place the `.tf` files inside just as you would do for a root module. Terraform can load modules either from local relative paths or from remote repositories; if a module will be re-used by lots of configurations you need to place it in its own version control repository. It’s important to keep module tree relatively flat to make it easier to re-use them in different combinations.

You should not write modules that are just thin wrappers around single other resource types. If you have trouble finding a name for your module that isn't the same as the main resource type inside it, that may be a sign that your module is not creating any new abstraction and so the module is adding unnecessary complexity. Just use the resource type directly in the calling module instead.

For every resource defined in a shared module, include at least one output that references the resource. Variables and outputs let you infer dependencies between modules and resources. Without any outputs, users cannot properly order your module in relation to their Terraform configurations.

Make sure that shared modules follow [SemVer v2.0.0](https://semver.org/spec/v2.0.0.html) when new versions are tagged or released.

Shared modules must not configure [providers or backends](https://developer.hashicorp.com/terraform/language/providers/configuration#provider-configuration-1). Instead, configure providers and backends in root modules.

> **Warning**
> A module containing its own provider configurations is not compatible with the `for_each`, `count`, and `depends_on` arguments that were introduced in Terraform v0.13. See [Providers Within Modules](https://developer.hashicorp.com/terraform/language/modules/develop/providers) for more information.

Although provider configurations are shared between modules, shared modules must also declare their own [provider requirements](https://developer.hashicorp.com/terraform/language/providers/requirements), so that Terraform can ensure that there is a single version of the provider that is compatible with all modules in the configuration and to specify the source address that serves as the global (module-agnostic) identifier for a provider. It doesn't, however, specify any of the configuration settings that determine what remote endpoints the provider will access, such as an AWS region.

For shared modules, define the minimum required provider versions in a [required_providers block](https://developer.hashicorp.com/terraform/language/modules/develop/providers#provider-version-constraints-in-modules) in the `versions.tf`.

To declare that a module requires particular versions of a specific provider, use a required_providers block inside a terraform block:

```terraform
terraform {
  required_version = ">= 1.0.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 4.0.0"
    }
  }
}
```

If a shared module only supports a specific version of a provider, use the pessimistic constraint operator:

```terraform
terraform {
  required_version = ">= 1.0.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}
```

> The example above allows only the rightmost version components to increment. For example, `~> 4.0` will allow installation of `4.57.1` and `4.67.0` but not `5.0.0`. Refer to the [version constraint syntax documentation](https://developer.hashicorp.com/terraform/language/expressions/version-constraints#version-constraint-syntax) for more information.

#### Module version pinning using Git commit hash

As of version [2.3.306](https://github.com/bridgecrewio/checkov/releases/tag/2.3.306), Checkov now checks if Terraform module sources use a commit hash ([CKV_TF_1](https://github.com/bridgecrewio/checkov/issues/5366)) and fail if they don't, as public registries are vulnerable to [Supply Chain Attacks](https://medium.com/boostsecurity/erosion-of-trust-unmasking-supply-chain-vulnerabilities-in-the-terraform-registry-2af48a7eb2). This means that when referencing another module on a public registry, a commit hash should be used instead of a tag to [Manually pin the Git commit hash by pointing directly to the underlying Git repo URL](https://developer.hashicorp.com/terraform/language/modules/sources#selecting-a-revision). For example:

```terraform
module "lambda" {
  source = "github.com/terraform-aws-modules/terraform-aws-lambda.git?ref=e78cdf1f82944897ca6e30d6489f43cf24539374" #--> v4.18.0

  ...

}
```

### Naming convention

- Resource meta names must be snake-cased and should be contextual to the resource being created. This practice ensures consistency with the naming convention for resource types, data source types, and other predefined values. This convention does not apply to name [arguments](https://www.terraform.io/docs/glossary#argument).
- To simplify references to a resource that is the only one of its type (for example, a single load balancer for an entire module), name the resource `main`.
- To differentiate resources of the same type from each other (for example, `primary` and `secondary`), provide meaningful resource names.
- Make resource names singular.
- In the resource name, don't repeat the resource type.
- Inputs, local variables, and outputs representing numeric values (e.g., disk size, RAM size) must be named with units (e.g., `ram_size_gb`). Naming variables with units makes the expected input unit clear for configuration maintainers.
- For units of storage, use binary unit prefixes (powers of 1024): `kibi`, `mebi`, `gibi`. For all other units of measurement, use decimal unit prefixes (powers of 1000): `kilo`, `mega`, `giga`.
- Give boolean variables positive names. For example, `enable_external_access`.

### Stateful Resources

For stateful resources, such as databases, ensure that [deletion protection](https://www.terraform.io/language/meta-arguments/lifecycle) is enabled.

### Built-In Formatting and Validation

All Terraform files must conform to the standards of [terraform fmt](https://developer.hashicorp.com/terraform/cli/commands/fmt).

Use [terraform validate](https://developer.hashicorp.com/terraform/cli/commands/validate) to verify the syntax and structure of your configuration.

### Expressions Complexity

Limit the complexity of any individual interpolated expressions. If many functions are needed in a single expression, consider splitting it out into multiple expressions by using [local values](https://www.terraform.io/docs/configuration/locals.html).

Never have more than one [ternary](https://developer.hashicorp.com/terraform/language/expressions/conditionals) operation in a single line. Instead, use multiple local values to build up the logic.

### Conditional Values

To instantiate a resource conditionally, use the [count](https://www.terraform.io/language/meta-arguments/count) meta-argument. For example: `count = length(var.some_value) == 0 ? 0 : 1
`

Be sparing when using user-specified variables to set the `count` variable for resources. If a resource attribute is provided for such a variable (like `project_id`) and that resource does not yet exist, Terraform can't generate a plan. Instead, Terraform reports the error [value of count cannot be computed](https://github.com/hashicorp/terraform/issues/17421). In such cases, use a separate `enable_x variable` to compute the conditional logic.

### Iterated Resources

Terraform can dynamically create resources using either [count](https://www.terraform.io/language/meta-arguments/count#the-count-meta-argument) or [for_each](https://www.terraform.io/language/meta-arguments/for_each). `for_each` should always be preferred over `count` except for circumstances where only count = 0 or 1 like explained in the [Conditional Values](#conditional-values) section. The reasoning for this comes from the behavior fundamental to lists vs maps; Lists are ordered; say you create 3 subnets [`subnet0`, `subnet1`, `subnet2`]. If you have to erase subnet 0 or 1, Terraform’s state file will see a change to the list and cause cascading unexpected changes. Using `for_each` resources are named using the map key.

For example with `aws_subnet.test[0].id` vs `aws_subnet.test["private_subnet0"].id`, you can delete `private_subnet0` without any fear of unintended consequences.

### Attachment Resources

Some resources have pseudo resources embedded as attributes in them. Where possible, you should avoid using these embedded resource attributes and instead you should use the unique resource to attach that pseudo-resource. These resource relationships can cause chicken/egg issues that are unique per resource.

Using embedded attribute (avoid this pattern):

```terraform
resource "aws_security_group" "allow_tls" {
  ...
  ingress {
    description      = "TLS from VPC"
    from_port        = 443
    to_port          = 443
    protocol         = "tcp"
    cidr_blocks      = [aws_vpc.main.cidr_block]
    ipv6_cidr_blocks = [aws_vpc.main.ipv6_cidr_block]
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }
}
```

Using attachment resources (preferred):

```terraform
resource "aws_security_group" "allow_tls" {
  ...
}

resource "aws_security_group_rule" "example" {
  type              = "ingress"
  description      = "TLS from VPC"
  from_port        = 443
  to_port          = 443
  protocol         = "tcp"
  cidr_blocks      = [aws_vpc.main.cidr_block]
  ipv6_cidr_blocks = [aws_vpc.main.ipv6_cidr_block]
  security_group_id = aws_security_group.allow_tls.id
}
```

### Custom Objects

Terraform allows you to create custom object types to constrain input that is allowed. For example:

```terraform
variable "safety_rules" {
  description = "Configuration of the Safety Rules. Key is the name applied to the rule."

  type = map(object({
    wait_period_ms = number
    inverted       = bool
    threshold      = number
    type           = string
    name_suffix    = string
  }))
}
```

### Workspaces

Workspace-separated environments use the same Terraform code but have different state files, which keep the environments as similar to each other as possible. Using workspaces organizes the resources in your state file by environments, so you only need one output value definition.

Define your environment specific variables for each environment using `.tfvars files` (non-sensitive) and [workspace variables](https://developer.hashicorp.com/terraform/cloud-docs/workspaces/variables) (sensitive).

### Remote state

[terraform_remote_state data source](https://developer.hashicorp.com/terraform/language/state/remote-state-data) must be used to retrieve the state outputs for a given workspace. It enables output values in one Terraform configuration to be used in another.

### Default Tags

All resource that can accept tags should. The terraform `aws` provider has a [default_tags](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/default_tags) feature that should be used inside the root module.

Consider adding necessary tags to all resources created by a Terraform module. Here is a list of possible tags to attach:

- **Name**: Human readable resource name.
- **AppId**: The application using the resource.
- **AppRole**: The resource’s technical function, e.g. "webserver", "database".
- **AppPurpose**: The resource’s business purpose, e.g. "frontend ui", "payment processor".
- **Environment**: dev, test or prod.
- **Project**: projects that use the resource.
- **CostCenter**: who to bill for the resource usage.

### Plan

Always generate a [plan](https://developer.hashicorp.com/terraform/cli/commands/plan) first for Terraform executions. After an owner approves it, execute the plan. When developers are locally prototyping changes, they should generate a plan and review the resources to be added, modified, and destroyed before applying the plan.

### Secrets

There are many resources and data providers in Terraform that store secret values in plaintext in the state file. Avoid storing secrets in state and use [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html) instead.

Instead of attempting to manually [encrypt sensitive values](https://www.terraform.io/docs/extend/best-practices/sensitive-state.html), rely on Terraform's built-in support for sensitive state management. When exporting sensitive values to output, make sure that the values are marked as [sensitive](https://www.terraform.io/docs/configuration/outputs.html#sensitive-suppressing-values-in-cli-output).

### Pre-Commit Hooks

Developers need fast feedback for potential issues with their code. Automation should run in their workspace to give them feedback before the deployment pipeline runs.

Pre-commit hooks are scripts that are executed on the local development environment before creating a new commit. These hooks allow to inspect the state of the code before the commit occurs and abort a commit if tests fail.

The following pre-commit hooks should exist on all repositories containing Terraform code:

- Out-of-the-box hooks
  - check-yaml
  - check-json
  - trailing-whitespace
  - check-added-large-files
  - check-executables-have-shebangs
  - check-shebang-scripts-are-executable
  - check-merge-conflict
  - check-vcs-permalinks
  - detect-private-key
  - detect-aws-credentials
  - end-of-file-fixer
  - no-commit-to-branch
  - pretty-format-json (autofix)
- gitleaks
- checkov
- pre-commit-terraform
  - terraform_fmt
  - terraform_validate
  - terraform_docs
  - terraform_tflint

> The standard pre-commit hooks configuration file is available in the [AWS Terraform template repository](https://github.com/aws-samples/aws-terraform-template/blob/main/.pre-commit-config.yaml)

### Infrastructure testing

Before releasing code to production, infrastructure testing is performed in a test environment to validate that the latest code is deployable and aligns with the approved architecture. This validation is done by first deploying the code and then performing sanity checks against the deployment. All actions performed in this stage should complete within 30 minutes to provide fast-feedback.

The tool for creating test environments and perform infrastructure tests is [Terratest](https://terratest.gruntwork.io/).

> A basic Terratest test file is available in the [AWS Terraform template repository](https://github.com/aws-samples/aws-terraform-template/blob/main/tests/infrastructure_tests/example_test.go)

## Terraform Module Creation

This section provides the steps required to create Terraform modules.

### Source Repository

A module repository must meet all of the following [requirements](https://developer.hashicorp.com/terraform/cloud-docs/registry/publish-modules#preparing-a-module-repository) so it can be published to a Terraform registry:

> You should always follow these requirements even if you are not planning to publish the module to a registry in the short term. This will allow the module to be published in a registry later without having to change the configuration and structure of the repository.

- Named `terraform-<PROVIDER>-<NAME>`: Module repositories must use this three-part name format, where `<NAME>` reflects the type of infrastructure the module manages and `<PROVIDER>` is the main provider where it creates that infrastructure. The `<PROVIDER>` segment must be all lowercase. The `<NAME>` segment can contain additional hyphens (e.g., `terraform-aws-iam-terraform-roles`).
- Standard module structure: The module must adhere to the [standard module structure](#standard-module-structure). This allows the registry to inspect your module and generate documentation, track resource usage, and more.

Once the GitHub repository has been created, the module files located need to be copied at the root of the repository. We recommend placing each module that is intended to be re-usable in the root of its own repository, but it is also possible to reference modules from subdirectories.

### Terraform Registry

Teams who are using Terraform Cloud/Enterprise should publish modules intended to be shared to their organization registry. The registry handles downloads and controls access with Terraform Cloud/Enterprise API tokens, so consumers do not need access to the module's source repository, even when running Terraform from the command line.

A module repository must meet all of the following [requirements](https://developer.hashicorp.com/terraform/cloud-docs/registry/publish-modules#preparing-a-module-repository) before it can be published to a Terraform registry:

- Location and permissions: The repository must be in one of your configured [VCS providers](https://developer.hashicorp.com/terraform/cloud-docs/vcs), and Terraform Cloud/Enterprise VCS user account must have admin access to the repository. The registry needs admin access to create the webhooks to import new module versions.
- x.y.z tags for releases: At least one release tag must be present for you to publish a module. The registry uses release tags to identify module versions. Release tag names must be a [semantic version](http://semver.org/), which you can optionally prefix with a `v` (e.g., `v1.1.0` and `1.1.0`). The registry ignores tags that do not look like version numbers.

> Refer to the [documentation](https://developer.hashicorp.com/terraform/cloud-docs/registry/publish-modules#publishing-a-new-module) for details about publishing modules.

### Module Sources

Terraform uses the source argument in a module block to find and download the source code for the desired child module.

The recommendation is to use local paths for closely-related modules used primarily for the purpose of factoring out repeated code elements, and using a native Terraform module registry or a VCS provider for modules intended to be shared by multiple configurations.

Examples of the most common and recommended [source types](https://developer.hashicorp.com/terraform/language/modules/sources) for sharing modules are provided in the following sections.

#### Registry

##### Hashicorp Terraform Registry

```terraform
module "lambda" {
  source = "github.com/terraform-aws-modules/terraform-aws-lambda.git?ref=e78cdf1f82944897ca6e30d6489f43cf24539374" #--> v4.18.0

  ...

}
```

##### Terraform Cloud

```terraform
module "eks" {
  source = "app.terraform.io/my-org/iam-terraform-roles/aws"
  version = "1.1.0"

  ...

}
```

##### Terraform Enterprise

```terraform
module "lambda_example_terraform_iam_roles" {
  source = "terraform.xom.cloud/my-org/iam-terraform-roles/aws"
  version = "1.1.0"

  ...

}
```

> Registry modules support [versioning](https://developer.hashicorp.com/terraform/language/modules/syntax#version). You should always provide a specific version as shown in the above examples.

#### VCS Providers

##### GitHub (HTTPS)

```terraform
module "lambda_example_terraform_iam_roles" {
  source = "github.com/my-org/terraform-aws-iam-terraform-roles.git?ref=v1.1.0"

  ...

}
```

##### Generic Git Repository (HTTPS)

```terraform
module "lambda_example_terraform_iam_roles" {
  source = "git::https://example.com/terraform-aws-iam-terraform-roles.git?ref=v1.1.0"

  ...

}
```

##### Generic Git Repository (SSH)

> **Warning**
>You will need to configure credentials to access private repositories.

```terraform
module "lambda_example_terraform_iam_roles" {
  source = "git::ssh://username@example.com/terraform-aws-iam-terraform-roles.git?ref=v1.1.0"

  ...

}
```

> VCS providers support the `ref` argument for selecting a specific revision as shown in the above examples.

## Terraform Backend Configuration

This section provides information about how to configure a backend to persist state data and to keep track of the resources Terraform manages.

When no backend configuration is provided, Terraform uses a backend called [local](https://developer.hashicorp.com/terraform/language/settings/backends/local), which stores state as a local file on disk. This is not ideal as this not allows multiple people to access the state data and work together on that collection of infrastructure resources.

A Terraform configurations should either integrate with Terraform Cloud/Enterprise or use a [backend](https://developer.hashicorp.com/terraform/language/settings/backends/configuration#available-backends) to store state remotely.

### Terraform Cloud/Enterprise Configuration

The main module of a Terraform configuration can integrate with Terraform Cloud/Enterprise to enable its CLI-driven run workflow. You only need to configure these settings when you want to use Terraform CLI to interact with Terraform Cloud/Enterprise. Terraform Cloud/Enterprise ignores them when interacting with Terraform through version control or the API.

To configure the Terraform Cloud CLI integration, add a nested [cloud block](https://developer.hashicorp.com/terraform/language/settings/terraform-cloud) within the terraform block.

```terraform
terraform {
  cloud {}
}
```

The most flexible approach is to use [Environment Variables](https://developer.hashicorp.com/terraform/cli/cloud/settings#environment-variables) along with [Environment Variable Credentials](https://developer.hashicorp.com/terraform/cli/config/config-file#environment-variable-credentials) to configure the cloud block attributes. This is helpful when you want to configure Terraform as part of a Continuous Integration (CI) pipeline. Terraform only reads these variables if the corresponding attribute is omitted from your configuration file.

This configuration is ideal for using [non-interactive workflows](https://developer.hashicorp.com/terraform/cloud-docs/run/cli#non-interactive-workflows) that minimize the risk of unpredictable infrastructure changes and configuration drift by ensuring that no one can change your infrastructure outside of your automated build pipeline.

> **Warning**
> You cannot use the CLI integration and a state backend in the same configuration.

### Amazon S3 Configuration

Stores the state as a given key in a given bucket on [Amazon S3](https://developer.hashicorp.com/terraform/language/settings/backends/s3). This backend also supports state locking and consistency checking via Dynamo DB, which can be enabled by setting the `dynamodb_table` field to an existing DynamoDB table name. A single DynamoDB table can be used to lock multiple remote state files. Terraform generates key names that include the values of the `bucket` and `key` variables.

```terraform
terraform {
  backend "s3" {
    bucket = "mybucket"
    key    = "path/to/my/key"
    region = "us-east-1"
  }
}
```


## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License Summary

The documentation is made available under the Creative Commons Attribution-ShareAlike 4.0 International License. See the LICENSE file.

The sample code within this documentation is made available under the MIT-0 license. See the LICENSE-SAMPLECODE file.
