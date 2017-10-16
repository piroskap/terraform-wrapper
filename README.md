Inspired by https://stackoverflow.com/a/45968410

# Installing

## Main script
Copy ``tf.sh somewhere``, ``/usr/local/bin/tf`` for instance.

## Bash prompt
Copy ``tf-prompt.sh`` somewhere, source it from your bash profile. 

The ``tf_prompt`` function returns ``(<workspace>/<stack>)``. Call `\$(tf_prompt)` somewhere in your prompt variables.

# Directory structure:

````
.tf/                        Internal configuration. Indicates the root directory.
    .workspace              Current workspace
    .stack                  Current stack
envvars/
    <workspace 1.tfvars>    Variables for <workspace 1>
    <workspace 2.tfvars>    Variables for <workspace 2>
    <...>
stacks/
    <stack 1>
        backend.tf          Partial backend configuration, only declar type (S3) and region
    <stack 2>
        backend.tf
    <...>
accounts                    Mapping between workspace names and AWS accounts
backend.tf                  Global terraform backend file. Will be copied or symlinked in each stack
global.tf                   Global terraform file. Will be copied or symlinked in each stack
global.tfvars               Global variables, shared by all stacks
````

# Workspaces

You can name your workspaces the way you want, but you need to have:
* a mapping in ``accounts``: ``<workspace>=<AWS account>``
* a AWS profile (in .aws/config and .aws/credentials) named ``<workspace>``
* a file in ``envvars`` named ``<workspace>``

# Pre-requisites

You must have a S3 bucket named ``terraform-state-<account id>`` in each account, and your profile must have:
* read access to the state files of the stacks you consume
* write access to the state files of the stacks you modify

You can use the scripts in ``state-management`` to create the buckets and read/write policies.
Policies are not assigned automatically.

``backend.tf`` should contain something like:
````
terraform {
  backend "s3" {
    region = "eu-west-1"
    encrypt = true
  }
}
````

# Starting up

Execute the script from a directory inside a root (i.e. one of the parent directory must contain a ``.tf`` folder).

Select a stack:
    
    tf stack network
    
Select a workspace:

    tf workspace acc
    
Do some stuff:

    tf plan
    tf apply

``global.tf``, ``global.tfvars`` and ``envvars/<workspace>.tfvars`` are automatically used when running:    
* ``apply`` 
* ``destroy``
* ``import``
* ``plan``
* ``refresh``
* ``validate`` 

``apply`` always use ``-auto-approve=false``. In an automation scenario, use ``./tf.sh apply -auto-approve=true``

# Unsupported terraform functions

    console
    env
    init
    providers
    push
    workspace

# TODO

1. Show current workspace and stack in bash prompt (in progress, see tf-prompt.sh)
1. Generate a graph of the stacks dependencies, relying on usage of ``terraform_remote_state`` in each stack
1. Bash autocompletion
1. Initialize a project.
1. Use symlinks if supported instead of copying ``global.tf`` as ``global.symlink.tf``
1. ? Find current stack from current directory, to be able to use ``cd stacks/xxx`` instead of ``tf stack xxx``