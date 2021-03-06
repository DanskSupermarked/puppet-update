# Update with Puppet
[![Build Status](https://travis-ci.org/DanskSupermarked/puppet-update.svg?branch=master)](https://travis-ci.org/DanskSupermarked/puppet-update)

Would like to know which packages should be updated on a node?

Would like that list of packages to be injected in your Puppet code?

## How it Works
The module will help you configure and schedule [update-with-puppet](https://github.com/DanskSupermarked/update-with-puppet).
By default the module will poll once GitHub for [update-with-puppet](https://github.com/DanskSupermarked/update-with-puppet), this feature is only supported since [Puppet 4.4](https://docs.puppet.com/puppet/latest/type.html#file-attribute-source). The 'file_src_base_uri' and 'file_replace' parameters can be overwritten to change that behavior and load the [three Python files](https://github.com/DanskSupermarked/update-with-puppet/tree/master/app) via one of the other supported Puppet sources.

The CRON job will be set for the day and hour defined, the minute of execution will be randomly generated to avoid running at the same time on multiple nodes in the same environment.

The job will fetch from the package provider the updates available for the specified package repositories.
A list of [Puppet Package](https://docs.puppet.com/puppet/latest/type.html#package) resource will be generated (For now as Hiera JSON).
The list will, by default, create the Package resources under a Hiera key named 'packages' which should be used in your site.pp or other top level Puppet file to use a lookup key to create resources.
This list will be committed to a GIT repository where your Puppet configuration is.
You're then free to have those Package resources updated by Puppet.

## Example
After assigning the update class to a node, the following parameters should be set.

### JSON Hiera
- With GIT commit and Pull Request creation.
```json
{
  "update::generate_pr": true,
  "update::git_account_name": "YOUR_GIT_ACCOUNT_WHERE_PUPPET_CONF_IS",
  "update::git_email": "THE_EMAIL_OF_THE_GIT_USER",
  "update::git_password": "GIT_USER_PASSWORD",
  "update::git_repo_name": "THE_GIT_REPO_NAME_WHERE_PUPPER_CONF_IS",
  "update::git_user": "GIT_USER",
  "update::git_username": "GIT_USERNAME (optional)",
  "update::hiera_file": "THE_HIERA_FILE_TO_WRITE_RESOURCE_TO (Fact in name is a good idea)",
  "update::pr_reviewers": "LIST_OF_PR_REVIEWERS (optional)",
  "update::repo_filter": "COMMA_SEPARATED_LIST_OF_YUM_REPO_TO_SEARCH",
  "update::working_branch": "NAME_OF_GIT_BRANCH_TO_COMMIT_TO"
}
```
- With GIT commit in a feature branch, no Pull Request.
```json
{
  "update::git_account_name": "YOUR_GIT_ACCOUNT_WHERE_PUPPET_CONF_IS",
  "update::git_email": "THE_EMAIL_OF_THE_GIT_USER",
  "update::git_password": "GIT_USER_PASSWORD",
  "update::git_repo_name": "THE_GIT_REPO_NAME_WHERE_PUPPER_CONF_IS",
  "update::git_user": "GIT_USER",
  "update::git_username": "GIT_USERNAME (optional)",
  "update::hiera_file": "THE_HIERA_FILE_TO_WRITE_RESOURCE_TO (Fact in name is a good idea)",
  "update::repo_filter": "COMMA_SEPARATED_LIST_OF_YUM_REPO_TO_SEARCH",
  "update::working_branch": "NAME_OF_GIT_BRANCH_TO_COMMIT_TO"
}
```

The module will attempt to install Python modules via RPM ('manage_python_deps' is set to true by default), one of them expects EPEL to be defined in Puppet as Yumrepo['epel']. You can also set the param to false and install those Python module via [pip](https://github.com/DanskSupermarked/update-with-puppet/blob/master/requirements.txt) as site libraries.

## Use Case
A [CRON](https://docs.puppet.com/puppet/latest/type.html#cron) job will collect a list of packages to be updated.
If a pull request with the defined 'pr_title' doesn't already exist, one can be created for data collected in that environemnt/branch.
Then create a GIT PR to be reviewed, eventually edited, and finally merged in your Puppet configuration to have the packages updated during the next Puppet run.

## OS Support
- RPM based Linux: RHEL, Centos, Scientific, older Fedora,...
- DNF based Linux: newer Fedora.

## Repository Support
- GIT.
- BitBucket API for pull request creation.

## When updating the module
The code managed by this module is by default loaded from GitHub only once. To force their upgrade, set 'file_replace' to true.

#### Copyright

Copyright 2017 [Dansk Supermarked Group](https://dansksupermarked.com/) and released under the terms of the [GPL version 3 license](https://www.gnu.org/licenses/gpl-3.0-standalone.html).
