# registry.terraform.io
## Repo to keep compiled plugins for darwin_arm64  - compiled from registry.terraform.io

This is a repo that can be directly cloned into 

**~/.terraform.d/plugins/**

I have compiled the required plugins for my requirements for my M1 Mac, feel free to compile your own if required.

Below is the full write up on why and how a single plugin was creted.

## Issue and Solution

I came across another issue while trying to run terragrunt init on my Mac. The arm_64 version of the mysql 1.9.0 plugin does not exist and was giving errors when running terragrunt init. The following is the exact error I received.
 
> │ Error: Incompatible provider version <br />
> │ <br />
> │ Provider registry.terraform.io/terraform-providers/mysql  v1.9.0 does not <br />
> │ have a package available for your current platform, darwin_arm64. <br />
> │ <br />
> │ Provider releases are separate from Terraform CLI releases, so not all <br />
> │ providers are available for all platforms. Other versions of this provider <br />
> │ may have different platforms supported. <br />
> ERRO[0039] 1 error occurred: <br />
> \* exit status 1 <br />
 
I could have used a newer, compatible version (where's the fun in that?), but I want to keep thing consistent with what is in the repo’s provider file, without having to change any code nor messing about with version differences of binaries - while at the same time, provide a fix for other plugins that may also not be compatible in the future. So, after a while of debugging compiling and recompiling I have finally found a solution worth a write out, all without altering any code in the terraform providers file in the GitHub repositories - theoretically, it should work with all other plugins that do not have a compatible arm_64 version.
Using GO, I compile the plugin from source (which was pulled from GitHub and branched to the correct version) on my M1 Mac.
The following was done in order:


1. git clone https://github.com/hashicorp/terraform-provider-mysql.git
2. cd terraform-provider-mysql
3. git checkout v1.9.0
4. go build
5. go install <br />

the resulting binary was then exported to the GO folder: ~/go/bin/terraform-provider-mysql
and then moved into the terraform local plugins dir, which by default is located at 
> ~/.terraform.d/plugins/

Numerous dir’s had to be created, the path which hold the binary followed a specific directory convention - plugin provider / plugin name / version / arch (in this case arm64) / binary file I.e.
> ~/.terraform.d/plugins/registry.terraform.io/terraform-providers/mysql/1.9.0/darwin_arm64

The thing is, when the binary is compiled, it exports a file with the default name (no versioning)
terraform-provider-mysql
This will not work if it copied as is, and a certain naming convention needs to be followed for this. So the binary was renamed to the following and placed into the above mentioned dir, I will include the full path (inc binary) at the bottom of the page.
terraform-provider-mysql_v1.9.0_v5
The following comment on GitHub was extremely helpful and played a big part in helping solve this issue: darwin/arm64 build · Issue #27257 · hashicorp/terraform 
After doing this and running terragrunt init again, I am met with the following:
> Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.
If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
:partying_face:  <br />
The full path of the mysql 1.9.0 dir and binary on my local machine:
 <br />
> ~/.terraform.d/plugins/registry.terraform.io/terraform-providers/mysql/1.9.0/darwin_arm64/terraform-provider-mysql_v1.9.0_v5
 <br />
It may also be good to note: <br />
After completing this, terragrunt plan also ran successfully and reported No changes.
Enabling terraform debug mode was also helpful in debugging any issue encountered throughout the process. <br \>

> TF_LOG=debug  
