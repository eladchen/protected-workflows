> Note:  
> - "anyEvent" property is omitted from the YAML examples for brevity  
> - "pull_request" is used in all the examples, but may be replaces or duplicated (recommended) by "push" & "pull_request_target" events

### Prevent changing mission critical files
Scenario:  
> I want to disallow the change of specific paths by anyone, except specific users / organisation members

Config:
```yaml
events:
  pull_request:
    - trustAnyone: true
      paths:
      	disallowed:
          - ".github/**"
          - "package.json"
          - "package-lock.json"
          - ".github/protected-workflows.yml" # <-- Exteremly important. Only certain users should be able to change this file.
    
    # Authorize "repo-owner" regardless of what paths were changed.
    - trustedUserNames:
        - "repo-owner"
```

### Allow specific users to change specific files
Scenario:
> I want to allow user "release-bot" to change CHANGELOG.md & package.json **only**

Config:
```yaml
events:
  pull_request:
    - trustedUserNames:
      - "release-bot"
      paths:
      	disallowed:
          - "CHANGELOG.md"
          - "package.json"
```

### Authorize just by the actor user name
Scenario:
> I want to allow users "repo-owner" & "repo-collaborator" to do anything

Config:
```yaml
events:
  pull_request:
    - trustedUserNames:
      - "repo-owner"
      - "repo-collaborator"
```

### Avoid repeating the same rules for two or more events
Scenario:
> I have two events, "pull_request" & "pull_request_target", and would like them to use the same rules

Config:
```yaml
events:
  # '&some-anchor-id' is a YAML anchor
  pull_request: &some-anchor-id
    - trustAnyone: true
      paths:
      	disallowed:
          - ".github/**"
          - "package.json"
    
  # '*some-anchor-id' is a YAML alias  
  pull_request_target: *some-anchor-id
```
Try out anchors & aliases online using https://onlineyamltools.com/convert-yaml-to-json
