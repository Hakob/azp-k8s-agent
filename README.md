## Before installation

Make sure that you are running on the host machine in order to be able azure agent work and share properly with internal directories like `workingDirectory` and so on.

If you have decided to run in `dind` mode you've probably need some additional setup for sharing volumes between agent and dind

```yaml
# <your>.values.yaml
agent:
  # ---------- AUTH  ----------
  # !!! This helm chart provides two way for passing the personal access token. PLEASE fill only 'pat' or 'patSecret'.
  # 1st way: Personal access token could be passed with 'pat' field.
  pat: ""

  # 2nd way: If a secret that contains personal access token exists in kubernetes, its name and key could be passed with following fields.
  patSecret: "azp-pat"
  patSecretKey: "AZP_TOKEN"
  # ---------- AUTH  ----------


  # Server / organization url, e.g.: https://dev.azure.com/your-organization-name
  organizationUrl: "https://dev.azure.com/{YOUR-ORG}"

  # Agent pool name which the build agent is placed into
  pool: "Default"

  # Working directory of agent
  workingDirectory: "/azp/_work"

  # Additional environment variables, if you need
  extraEnv:
    DOCKER_HOST: tcp://localhost:2375

nameOverride: ""
fullnameOverride: ""
replicaCount: 1

image:
  registry: docker.io
  repository: hakob/azp-k8s-agent
  pullPolicy: IfNotPresent
  tag: "20240403.1"
  pullSecrets: []

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi

volumes:
  - name: storage
    emptyDir: {}
  - name: azp-workdir
    emptyDir: {}

volumeMounts:
  - name: azp-workdir
    mountPath: /azp/_work
    subPath: _work

# Additional init containers, e. g. for providing custom themes
extraContainers:
    |
    - name: dind
      image: "docker:dind"
      imagePullPolicy: Always
      command: ["dockerd", "--host", "tcp://localhost:2375"]
      env:
        - name: DOCKER_TLS_CERTDIR
          value: ""
        - name: DOCKER_TLS_VERIFY
          value: "0"
        - name: DOCKER_HOST
          value: "tcp://localhost:2375"
      securityContext:
        privileged: true
      resources:
        limits:
          cpu: 200m
          memory: 256Mi
        requests:
          cpu: 100m
          memory: 128Mi
      volumeMounts:
        - name: storage
          mountPath: /var/lib/docker
          subPath: docker
        - name: azp-workdir
          mountPath: /azp/_work
          subPath: _work

serviceAccount:
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: ""
  # Specifies whether a service account should be created
  create: true
  # Automatically mount a ServiceAccount's API credentials?
  automount: true
  # Annotations to add to the service account
  annotations: {}

nodeSelector: {}
tolerations: []
affinity: {}
podLabels: {}
podAnnotations: {}
podSecurityContext: {}
securityContext: {}
```

## Installing the Chart


1. First you need to add repository _(if you haven't done yet before)_
```bash
helm repo add hakob https://hakob.github.io
```

2. Install the helm chart with required parameters
  - With bash:
```bash
helm install {RELEASE-NAME} hakob/azure-devops-agent \
  --set agent.pat={PAT} \
  --set agent.organizationUrl=https://dev.azure.com/{YOUR-ORG} \
  --namespace {YOUR-NS}
```

  - With powershell:
```powershell
helm install {RELEASE-NAME} hakob/azure-devops-agent `
  --set agent.pat={PAT} `
  --set agent.organizationUrl=https://dev.azure.com/{YOUR-ORG} `
  --namespace {YOUR-NS}
```

## Uninstalling the Chart

Run the following snippet to uninstall the release:
```bash
helm delete {RELEASE-NAME}
```

## Parameters

### Agent authentication parameters

> :warning: Helm chart provides two option for authentication. Please use only one of them.

| Name                 | Description                                                       | Value   |
| -------------------- | ----------------------------------------------------------------- | ------- |
| `agent.pat`          | (1st Option) Personal access token for authentication             | `""`    |
| `agent.patSecret`    | (2nd Option) Already existing secret name that stores PAT         | `""`    |
| `agent.patSecretKey` | (2nd Option) Key (field) name of the PAT that is stored in secret | `"pat"` |


### Agent configuration parameters

| Name                     | Description                                                                   | Value       |
| ------------------------ | ----------------------------------------------------------------------------- | ----------- |
| `agent.organizationUrl`  | Server / organization url, e.g.: https://dev.azure.com/your-organization-name | `""`        |
| `agent.pool`             | Agent pool name which the build agent is placed into                          | `"Default"` |
| `agent.workingDirectory` | Working directory of the agent                                                | `"_work"`   |
| `agent.extraEnv`         | Additional environment variables as dictionary                                | `{}`        |

### Other parameters

| Name                        | Description                                                   | Value                 |
| --------------------------- | ------------------------------------------------------------- | --------------------- |
| `image.repository`          | Azure DevOps agent image repository                           | `hakob/azp-k8s-agent` |
| `image.tag`                 | Azure DevOps agent image tag (immutable tags are recommended) | `3.232.3`             |
| `image.pullPolicy`          | Azure DevOps agent image pull policy                          | `IfNotPresent`        |
| `image.pullSecrets`         | Azure DevOps agent image pull secrets                         | `[]`                  |
| `replicaCount`              | Replica count for deployment                                  | `1`                   |
| `resources.requests.cpu`    | CPU request value for scheduling                              | `"100m"`              |
| `resources.requests.memory` | Memory request value for scheduling                           | `"128Mi"`             |
| `resources.limits.cpu`      | CPU limit value for scheduling                                | `"500m"`              |
| `resources.limits.memory`   | Memory limit value for scheduling                             | `"512Mi"`             |
| `volumes`                   | Volumes for the container                                     | `[]`                  |
| `volumeMounts`              | Volume mountings                                              | `[]`                  |

Please refer the values.yaml for other parameters.

## Built-in binaries & packages
The binaries and packages listed below are included in the docker image used by the helm chart:
- Ubuntu 22.04
- unzip
- jq
- yq
- git
- helm
- kubectl
- Powershell Core
- Docker CLI

## References

For the original chart please refer to [Burak Tungut's](https://github.com/btungut) repositories
