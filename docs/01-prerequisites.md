# Prerequisites

In this lab you will review the machine requirements necessary to follow this tutorial.

## Virtual or Physical Machines

This tutorial requires four (4) virtual or physical ARM64 machines running Debian 12 (bookworm). The follow table list the four machines and thier CPU, memory, and storage requirements.

| Name    | Description            | CPU | RAM   | Storage |
|---------|------------------------|-----|-------|---------|
| jumpbox | Administration host    | 2   | 2GB   | 20GB    |
| server  | Kubernetes server      | 2   | 2GB   | 20GB    |
| node-0  | Kubernetes worker node | 2   | 2GB   | 20GB    |
| node-1  | Kubernetes worker node | 2   | 2GB   | 20GB    |

How you provision the machines is up to you, the only requirement is that each machine meet the above system requirements including the machine specs and OS version. Once you have all four machine provisioned, verify the system requirements by running the `uname` command on each machine:

```bash 
uname -mov
```

After running the `uname` command you should see the following output:

```text
#115-Ubuntu SMP Mon Apr 15 09:52:04 UTC 2024 x86_64 GNU/Linux
```

Next: [setting-up-the-jumpbox](02-jumpbox.md)
