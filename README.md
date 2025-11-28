# ModSecurity Helm Chart with OWASP Core Rule Set (CRS)

## Overview
This Helm chart deploys **ModSecurity** with the **[official OWASP Core Rule Set (CRS) Docker container](https://hub.docker.com/r/owasp/modsecurity-crs)**, allowing users to integrate a Web Application Firewall (WAF) into their infrastructure. The setup enables ModSecurity to function as a reverse proxy, sitting between an Ingress controller and an application to filter and block malicious traffic.

## Features
- Deploys **ModSecurity** using the official **[OWASP CRS Docker container](https://hub.docker.com/r/owasp/modsecurity-crs)**
- Easily integrates into existing Kubernetes environments
- Configurable rule set and logging options
- Deployable as a standalone proxy or in conjunction with an Ingress controller
- Supports custom ModSecurity configurations
- Supports **GeoIP Blocking** via DB-IP or MaxMind GeoLite2 database

## OWASP CRS version
The chart deploys the latest v4.x version (see chart's appVersion and `image.tag` value) of OWASP's CRS Apache container.
The Apache container is based on ModSecurity v2.9.x for [better stability and performance](https://coreruleset.org/docs/1-getting-started/1-4-engine_integration_options/#modsecurity-v2).

## Prerequisites
- A running **Kubernetes cluster**
- **Helm** installed ([Installation Guide](https://helm.sh/docs/intro/install/))
- An existing **Ingress Controller** (optional, if using ModSecurity as an additional WAF layer)
- **MaxMind GeoLite2 account** (optional, for GeoIP-based blocking)

## Installation

### 1. Install the Chart
```sh
helm install --create-namespace -n your-namespace modsecurity-crs-proxy oci://registry-1.docker.io/phoenixmedia/modsecurity-crs-proxy --version 0.1.0 -f values.yaml
```

Alternatively, install with default settings:
```sh
helm install --create-namespace -n your-namespace --set BACKEND=http://my-app-service modsecurity-crs-proxy oci://registry-1.docker.io/phoenixmedia/modsecurity-crs-proxy
```

#### OpenShift Deployment

When deploying on **OpenShift**, additional configuration is required due to Security Context Constraints (SCCs). The ModSecurity container runs as a non-root user (UID 999), which requires binding the ServiceAccount to an appropriate SCC.

**Prerequisites:**
This chart automatically creates a ServiceAccount and configures the pod to run as UID 999. You only need to bind the ServiceAccount to a compatible SCC.

**SCC Configuration:**
After installing the chart, bind the ServiceAccount to the `nonroot-v2` SCC (or any SCC that allows running as UID 999):

```sh
# Get the ServiceAccount name (typically: <release-name>-modsecurity-crs-proxy)
SA_NAME=$(kubectl get sa -n your-namespace -l app.kubernetes.io/name=modsecurity-crs-proxy -o jsonpath='{.items[0].metadata.name}')

# Bind to nonroot-v2 SCC
oc adm policy add-scc-to-user nonroot-v2 -z $SA_NAME -n your-namespace
```

Alternatively, if you know your release name:
```sh
oc adm policy add-scc-to-user nonroot-v2 -z modsecurity-crs-proxy -n your-namespace
```

**Security Context:**
The chart is pre-configured with the following security settings for OpenShift compatibility:
- `runAsUser: 999` - Runs as the httpd user
- `runAsGroup: 999` - Uses the httpd group
- `fsGroup: 999` - Sets filesystem permissions
- `runAsNonRoot: true` - Enforces non-root execution
- `allowPrivilegeEscalation: false` - Prevents privilege escalation
- Drops all Linux capabilities for enhanced security

These settings are defined in `values.yaml` and can be customized if needed.

### 2. Verify Deployment
Check if the pods are running:
```sh
kubectl get pods -n your-namespace
```

## GeoIP Blocking
This Helm chart supports **GeoIP-based blocking** using the **DB-IP** or **MaxMind GeoLite2-Country database**.
To enable checkout one of the `values_geoip_*.yaml` example files and set the required values.

## Configuration
This Helm chart allows customization through the `values.yaml` file. Below are some key configurable parameters:

| Parameter                  | Description                                          | Default |
|----------------------------|------------------------------------------------------|---------|
| `image.repository`         | The Docker image repository for ModSecurity with CRS | `owasp/modsecurity-crs` |
| `image.tag`                | The image tag to use (see chart appVersion)          | `` |
| `backup`                   | URL of the backend service                           | `http://my-app-service.svc.cluster.local:80` |
| `securityRules` | Custom ModSecurity settings and rules                | `[]` |
| `service.port`             | Port for the ModSecurity proxy                       | `8080` |
| `geoip.enabled` | Enable GeoIP-based blocking                          | `false` |

For a full list of configurable parameters, see the `values.yaml` file.

## Example Use Cases

### Deploying ModSecurity as a Reverse Proxy
To deploy ModSecurity as a standalone reverse proxy filtering traffic before reaching your application, use:
```yaml
backend: "http://my-app-service.svc.cluster.local:80"
```

This ensures that ModSecurity processes traffic before reaching your backend services.


### Disable disruptive action

ModSecurity supports [three engine modes](https://github.com/owasp-modsecurity/ModSecurity/wiki/Reference-Manual-%28v2.x%29#secruleengine):
"On", "Off" and "DetectionOnly". Modify the environment variable to change the setting:

```yaml
env:
  - name: MODSEC_RULE_ENGINE
    value: "DetectionOnly"
```
*Note: In order to modify a single environment variable you have to copy the whole env block from the `values.yaml`*

## Uninstallation
To remove the Helm deployment, run:
```sh
helm uninstall modsecurity-crs-proxy
```

## License
This Helm chart is released under the **Apache 2.0 License**. The official OWASP CRS container follows the same licensing terms.

## Contributing
Contributions are welcome! Please open an issue or submit a pull request on [GitHub](https://github.com/PHOENIX-MEDIA/modsecurity-crs-proxy).

## References
- [ModSecurity Documentation](https://github.com/owasp-modsecurity/ModSecurity)
- [OWASP CRS Documentation](https://coreruleset.org/)
- [OWASP CRS Container documentation](https://hub.docker.com/r/owasp/modsecurity-crs)
- [Helm Documentation](https://helm.sh/docs/)
- [MaxMind GeoIP Databases](https://www.maxmind.com/en/geoip2-services-and-databases)
