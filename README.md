# Protected Workflows
Protected Workflows is a [Github application](https://github.com/apps/protected-workflows). Its goal is to cancel [unauthorized](#when-is-a-workflow-run-unauthorized) workflow runs.

## Caveat:
> **Github has what I believe to be a bug in their design**:  
> Anyone with push access to a repository may rerun workflow jobs, which may be ok, except for the fact  
> that such events will not be relayed to this application.
>
> In other words, the above means that Protected-Workflows is unable to cancel unauthorized workflow runs when a workflow is reran.
>
> This bug has been reported to Github, and until it is addressed, its best to rely on this application
> to authorize pull requests (using the "pull_request" / "pull_request_target" events)

## Table of Contents
* [What Is the Purpose of This Repository](#what-is-the-purpose-of-this-repository)
* [Features](#features)
* [How Does This Application Work](#how-does-it-work)
  * [When is a Workflow Run "Unauthorized"?](#when-is-a-workflow-run-unauthorized)
* [Configuration](#configuration)
  * [Rule](#rule)
  * [Best Practices](#best-practices)
* [Usage](#usage)
* [Examples & Scenarios](/examples-and-scenarios.md)

## What Is the Purpose of This Repository
This repository is the offical place to view the documentation and report issues regarding [Protected-Workflows](https://github.com/apps/protected-workflows).

## Features
1. Cancel [unauthorized](#when-is-a-workflow-run-unauthorized) workflow runs
2. Write configuration using YAML **with anchors & aliases support**.
3. Combine and mix the following parameters to authorize workflow runs:
	1. The user name
	2. Whether the triggering user is:
		- A [collaborator](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-your-github-user-account/inviting-collaborators-to-a-personal-repository) (requires "admin" or "write" permission).
		- A member of the organisation (If the repository is owned by an organisation)
		- **Coming Soon** - A member of a certain team (**Note: teams feature is limited to organisation/enterprise repositories**)
	3. Changed files paths (**Limited to workflow runs triggered by "push", "pull_request" or "pull_request_target"**)

## How Does This Application Work
Whenever a workflow run is triggered the application will be notified by github, and will use the application  
[configuration](#configuration) to authorize the workflow run. Unauthorized workflow runs will be canceled.

### When Is a Workflow Run "Unauthorized"?
When not a single [rule](#rule) authorized the workflow run.

## Configuration
Configuration is to be written to a file using YAML, and stored within the
repository root under `.github/protected-workflows.yml`.

The YAML may contain two top level properties:
 - "events" which is a map between [Github event names](https://docs.github.com/en/actions/reference/events-that-trigger-workflows) and rules
 - "anyEvent" which will be used as a fallback rule when event specific rules are undefined 

### Rule
A "Rule" is an object made up of the following properties:

<table>
    <tr>
	<td><b>Property</b></td>
	<td><b>Description</b></td>
	<td><b>Type</b></td>
	<td><b>Default</b></td>
    </tr>
    <tr>
        <td>trustAnyone</td>
        <td>Authorize workflow runs triggered by any actor</td>
        <td>boolean</td>
        <td>false</td>
    </tr>
    <tr>
        <td>trustCollaborators</td>
        <td>Authorize workflow runs triggered by collaborators</td>
        <td>boolean</td>
        <td>false</td>
    </tr>
    <tr>
        <td>trustOrgMembers</td>
        <td>Authorize workflow runs triggered by organisation members</td>
        <td>boolean</td>
        <td>false</td>
    </tr>	
    <tr>
        <td>trustedUserNames</td>
        <td>Authorize workflow runs triggered by predefined user names</td>
        <td>string[]</td>
        <td>[]</td>
    </tr>	
    <tr>
        <td>paths</td>
        <td>Authorize workflow runs by changed paths. Can only be used with "push", "pull_request" and "pull_request_target" events</td>
        <td>Paths</td>
        <td><a href="#user-content-rule-paths">Paths</a></td>
    </tr>
</table>

<a id="rule-paths"></a>
Paths:

| **Property** | **Description**                                        | **Type** | **Default** |
|:------------:|:-------------------------------------------------------|:--------:|:-----------:|
| allowed      | What paths may be changed                              | string[] | []          |
| disallowed   | What paths may not change. Glob patterns are supported | string[] | []          |

Complete example:

```yaml
# "events" is a map between Github events and rules.
# possible event names can be seen at https://docs.github.com/en/actions/reference/events-that-trigger-workflows
events:
  # 'pull_request' is the Github event name.
  # '&pull_request' is a YAML anchor
  pull_request: &pull_request
  
    # Authorize any user when package.json or anything under .github folder was not changed.
    - trustAnyone: true
      paths:
      	disallowed:
          - ".github/**"
          - "package.json"
    
    # Authorize "bot" user when CHANGELOG.md is the only changed file.
    - trustedUserNames:
        - "bot"
      paths:
        allowed:
          - "CHANGELOG.md"

    # Authorize collaborators when package.json is the only changed file.
    - trustCollaborators: true
      paths:
        allowed:
          - "package.json"

  # Reference the "pull_request" anchor to reuse its configuration
  # Read about "pull_request_target" in this blog post:
  # https://github.blog/2020-08-03-github-actions-improvements-for-fork-and-pull-request-workflows/
  pull_request_target: *pull_request

# 'anyEvent' value is a rule, and will be used when an event specific configuration is not set.
# It is automatically added in case it was not explictly set and it does not supports the 'paths' property.
anyEvent:
  trustAnyone: false
  trustCollaborators: false
  trustedUserNames: []
```

> Attention: **Workflow runs will be cancelled only if all rules deemed the workflow is unauthorized**

### Best Practices
- Make sure only specific users are allowed to change the configuration file: .github/protected-workflows.yml. [Example](https://github.com/eladchen/protected-workflows/blob/main/examples-and-scenarios.md#prevent-changing-mission-critical-files)
- Use the same rules for "pull_request", "pull_request_target" & "push". (Avoid repeating yourself by using [anchors and aliases](https://github.com/eladchen/protected-workflows/blob/main/examples-and-scenarios.md#avoid-repeating-the-same-rules-for-two-or-more-events))
- Identify build, release and dependency manifest files and limit who can change them to specific users  

  Example of what such files may be are:
  - Files under the .github/workflows directory
  - Release Scripts
  - A Dockerfile
  - Dependencies files (package.json, package-lock.json, build.gradle etc...)

## Usage
1. [Install the app](https://github.com/apps/protected-workflows)
2. Create a configuration file using the above guidelines and store it within the repository root under `.github/protected-workflows.yml`
