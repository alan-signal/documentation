---
title: Getting Started with Cloud Workload Security
kind: documentation
description: "Follow the setup and installation instructions to get started with Runtime Security."
further_reading:
- link: "https://www.datadoghq.com/blog/datadog-runtime-security/"
  tag: "Blog"
  text: "Learn more about Datadog Runtime Security"
---

## Overview

Cloud Workload Security (CWS) provides real-time eBPF-powered threat detection across your hosts and containers. CWS is built directly into the Datadog Agent. Simply activate cloud workload security on your Datadog agents. No additional agents required and no additional complexity for your operations team.

CWS can monitor the following types of activity for suspicious, malicious, or anomalous behavior:
1. **File Integrity Monitoring** to watch for changes to key files and directories on hosts or containers in real time.
2. **Process Execution Monitoring** to monitor process executions for malicious activity on hosts or containers in real time.

## Requirements

* Datadog Agent >= 7.27.0
* Hosts/containers must be running Linux with kernel versions >= 4.15
  * All major Linux distributions running on those kernels are supported.
  * Custom kernel builds are not supported.

## Installation

{{< tabs >}}
{{% tab "Kubernetes" %}}

Follow the Helm instructions on this page with the following changes in your values.yaml:

**Note**: By enabling runtime capabilities in the Runtime Security Agent, the Datadog `system-probe` will also be activated automatically. The `system-probe` is required to collect FIM events.

{{< code-block lang="yaml" filename="values.yaml" >}}

datadog:
  ...
  securityAgent:
    runtime:
      enabled: true
 ...
agents:
  image:
    repository: datadog/agent
    tag: 7-jmx
    doNotCheckTag: true

{{< /code-block >}}

{{% /tab %}}

{{% tab "Docker" %}}

The following command can be used to start the Runtime Security Agent and `system-probe` in a Docker environment:

{{< code-block lang="bash" filename="docker-runtime-security.sh" >}}

DOCKER_CONTENT_TRUST=1 \
  docker run -d --name dd-agent \
  --security-opt apparmor:unconfined \
  --cap-add SYS_ADMIN \
  --cap-add SYS_RESOURCE \
  --cap-add SYS_PTRACE \
  --cap-add NET_ADMIN \
  --cap-add IPC_LOCK \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v /proc/:/host/proc/:ro \
  -v /sys/fs/cgroup/:/host/sys/fs/cgroup:ro \
  -v /etc/passwd:/etc/passwd:ro \
  -v /etc/group:/etc/group:ro \
  -v /:/host/root:ro \
  -v /etc/os-release:/host/etc/os-release \
  -v /sys/kernel/debug:/sys/kernel/debug \
  -e DD_RUNTIME_SECURITY_CONFIG_ENABLED=true \
  -e DD_SYSTEM_PROBE_ENABLED=true \
  -e HOST_ROOT=/host/root \
  -e DD_API_KEY=<API KEY> datadog/agent:7-jmx

{{< /code-block >}}

{{% /tab %}}

{{% tab "Debian" %}}

For a package-based deployment, the Datadog package has to be deployed: run `dkpg -i datadog-agent_7….deb`

By default Runtime Security is disabled. In order to enable it, both the datadog.yaml and the system-probe.yaml files need to be adapted. Run the following commands to enable these configurations:

{{< code-block lang="bash" filename="debian-runtime-security.sh" >}}

echo "runtime_security_config.enabled: true" > /etc/datadog-agent/security-agent.yaml

systemctl restart datadog-agent

{{< /code-block >}}

Both the security-agent and the system-probe have to be restarted once these changes have been applied.

{{% /tab %}}

{{% tab "Fedora/CentOS" %}}

For a package-based deployment, the Datadog package has to be deployed: run `yum/dnf install datadog-agent_7….rpm`

{{< code-block lang="bash" filename="fedora-centos-runtime-security.sh" >}}

echo "runtime_security_config.enabled: true" > /etc/datadog-agent/security-agent.yaml

systemctl restart datadog-agent

{{< /code-block >}}

{{% /tab %}}
{{< /tabs >}}

## Further Reading
{{< partial name="whats-next/whats-next.html" >}}
