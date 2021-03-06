---
layout: wmt/docs
title:  Tekton Task
side-navigation: wmt/docs-navigation.html
---

# {{ page.title }}

Concord supports interaction with the cloud infrastructure provisioning tool
Tekton with the `tekton` task as part of any flow.

- [Usage](#usage)
- [Parameters](#parameters)
- [Templates](#templates)
- [Examples](#examples)

## Usage

To be able to use the task in a Concord flow, it must be added as a
[dependency](../getting-started/concord-dsl.html#dependencies):

```yaml
configuration:
  dependencies:
  - mvn://com.walmartlabs.concord.plugins:tekton-task:{{ site.concord_plugins_version }}
```

This adds the task to the classpath and allows you to invoke it in any flow.

## Parameters

All Tekton operations support common parameters. The operation itself is
specified with the `action` parameter. Operation specific parameters are
separated into their relevant sections below by action name.

__Common Paramters__

- `action`: The action to perform with Tekton.
  - `provision`: [Provisions](#provisioning) new VMs.
  - `getAddresses`: Returns the [addresses or hostnames](#addresses) of VMs in
    the `addresses` variable, to be passed into other flows, such as deployment.
  - `replace`: [Replaces](#replace) existing VMs with new VMs.
  - `destroy`: [Destroys](#destroy) a given environment completely.
  - `getKeypair` Gets the SSH [keypair](#keypair) generated by Tekton for
    administrative actions such as application deployment.
  - `reboot`: [Reboots](#reboot) VMs.
  - `getInventory`: Returns the Tekton [inventory](#inventory) for a given
    environment.
  - `getCert`: Gets the [certificate](#certificate) for a given FQDN.
- `apiToken`: The Tekton API token to call Tekton with. A default one is
  typically configured globally, and users should normally not set this unless
  specifically asked or requested.
- `apiUrl`: The URL of the Tekton instance you will be calling. A default one is
  typically configured globally, and users should normally not set this unless
  specifically asked or requested.
- `envName`: The environment name to use with tekton, for example `dev`, `prod`, `qa`, `stg`, etc.

__Additional Parameters for `provision`__

- `templatePath`: Defaults to `compute.yml`. This is the location of your [template](#templates).

__Additional Parameters for `replace` and `reboot`__

- `targets`: A comma separated list of clouds and/or IP addresses to replace.

__Additional Parameters for `getCert`__

- `fqdn`: The FQDN who's cert we're getting
- `teamDL`: The DL of the team this cert belongs to
- `keyPassword`: The password for the certificate key
- `format`: The certificate format

## Templates

### Compute

A compute template tells Tekton what VMs to deploy where, and how many you would
like. They are written in YAML format.

```yaml
clouds:
  cdc:
    - zone: abc14
      computes: 2
      primary: true
      deploymentOrder: 0
  dfw:
    - zone: def14
      computes: 2
      primary: true
      deploymentOrder: 1
compute:
  entity: my-org
  name: test
  type: small
  image: centos-7.3
  user: app
  networkSecurityGroup:
    rules:
    - protocol: TCP
      port: 22
    - protocol: TCP
      port: 8080
    - protocol: TCP
      port: 80
      port: 9000
loadBalancer:
  loadBalancerMethod: ROUNDROBIN
  loadBalancerRules:
  - name: db-lb
    healthCheckPath: "GET /test"
    frontendRule:
      protocol: HTTP
      port: 80
    backendRule:
      protocol: HTTP
      port: 8080
gslb:
  loadBalancerMethod: PROXIMITY
  fqdn:
    fqdnName: tekton.examples.java-app.dev.example.com
```

## Examples

### Provisioning

Tekton can provision new VMs with the `provision` action. The VM characteristics
need to be configured in a [compute template](#compute). A template file named
`compute.yml` is automatically used by default. Alternative names can be
specified with the `templatePath` parameter.

```yaml
flows:
  default:
  - task: tekton
    in:
      action: provision
      envName: dev
  - log: ${addresses}
```
    
Provisioning results are printed to the process log, and also available via the
variable `addresses` for further operations.
