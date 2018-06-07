# ibm-container-resource
A concourse resource for managing [IBM Container Service](https://console.bluemix.net/docs/containers/cs_tech.html#ibm-cloud-kubernetes-service-technology) version number.

## Source Configuration

* `uri`: *Optional.* The IBM Container Service versions URI. If not specified, the resource will use `https://containers.bluemix.net/v1/kube-versions`. If specified, it overwrites `environment` setting.

* `environment`: *Optional.* The environment of IBM Container Service. If not specified, the resource will check version of production environment.

* `type`: *Optional. Default `default`.* If not specified, the resource will check default version.

* `version`: *Optional.* The version number to check. If specified, the resource will only detect version that greater than or equal this version.

### Example

Resource configuration for a staging environment of IBM Container Service:

``` yaml
resource_types:
- name: ibm-container
  type: docker-image
  source:
    repository: bluebosh/ibm-container-resource
    tag: latest

resources:
- name: ibm-container-stage
  type: ibm-container
  source:
    environment: stage1
    version: 1.7.16

jobs:
- name: check-ibm-container-version
  plan:
  - aggregate:
    - get: ibm-container-stage
      trigger: true
  - task: get-ibm-container-version
    config:
      platform: linux
      image_resource: { type: docker-image, source: { repository: bluebosh/ibm-container-resource, tag: latest } }
      inputs:
      - name: ibm-container-stage
      run:
        path: /bin/bash
        args:
        - -c
        - |
          #!/bin/bash
          cat ibm-container-stage/number

```

## Behavior

### `check`: Report the current version number.

Detects new version by fetching from the container service endpoint. If the `type` is latest, it returns latest version. If the `type` is oldest, it returns oldest version. If the `type` is other value, it returns default version. And this version should be equal to or greater than the `version`, otherwise it returns no version.

### `in`: Provide the version as a file.

Provides the version number to the build as a `version` file in the destination.

Can be configured to bump the version locally, which can be useful for getting
the `final` version ahead of time when building artifacts.
