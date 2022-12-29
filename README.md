# hocs-deployments-notprod

This repository controls deployments to notprod environments

## Deploy to QA

To deploy the latest code to QA

1. Go to the Actions tab
2. Click on 'Update QA versions yml'
3. Click on 'Run Workflow' and wait for the job to complete
4. Go to the Pull Requests tab
5. Click on the pull request and press 'Rebase and merge'.


## Deploy to Other environments

Manually update the versions.yaml file in the relevant folder
e.g. environments/qa/versions.yaml to deploy to demo.
This uses the drone tree plugin


## How to add a chart

To add a chart:

1. Ensure the repository is present, DECS projects should use the existing repo

charts.yaml:
```yaml
repositories:
  hocs-helm-charts: https://ukhomeoffice.github.io/hocs-helm-charts
```

2. Reference the chart

charts.yaml:
```yaml
  hocs-audit:
    name: hocs-audit
    deploy: false
    chart: hocs-helm-charts/hocs-audit
    version: ^4.0.0
    <<: *default
```

3. For each folder in the `environments` folder:

Add the repository name to versions.yaml
   
versions.yaml:
```yaml
versions:
  hocs-audit: 2.18.3
```

Add a `.yaml.gotmpl` file referencing the version with sensible values for the chart

hocs-audit.yaml.gotmpl:
```yaml
---
hocs-generic-service:
  version: {{ index .Values.versions "hocs-audit" }}

  deployment:
    annotations:
      downscaler/downtime: "Mon-Sun 00:00-07:50 Europe/London,Mon-Sun 18:05-24:00 Europe/London"
...
```

## Testing locally

you can run diff to understand changes before they are made:
```shell
helmfile diff --environment hocs-qa -n hocs-qa
```

It is possible to run
```shell
helmfile apply --interactive --environment hocs-qa -n hocs-qa
```
to do deployments, but they should typically be done through Drone to ensure the correct secret values are used.
Valid exceptions are changes where manual intervention is needed. e.g. breaking changes to network policies.
