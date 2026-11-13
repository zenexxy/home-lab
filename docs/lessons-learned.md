# Challenges and lessons learned

This page records practical design lessons from building and migrating the
homelab. It distinguishes configuration decisions from work that still needs
to be revalidated.

## 1. Networking a VM over a Wi-Fi uplink

**Challenge:** The Proxmox host used Wi-Fi as its upstream connection. A
traditional bridge to a wireless client interface is not equivalent to
bridging a wired Ethernet interface and can be unreliable or unsupported,
depending on the wireless hardware and access point.

**Approach:** Put the VM on an internal bridge (`vmbr1`) and route/NAT its
private subnet through the host's Wi-Fi interface.

**Lesson:** Choose the virtual network design based on the physical uplink.
For a Wi-Fi-only host, a routed private bridge is often a more practical
starting point than trying to make the guest appear directly on the wireless
LAN. Recheck forwarding, firewall rules, DNS, and interface names after a
reinstall.

## 2. Keeping application deployment reproducible

**Challenge:** Recreating multiple self-hosted applications by hand makes it
easy to forget environment variables, volumes, ports, and dependencies.

**Approach:** Use Docker Compose definitions to describe the services and
their persistent mounts. Keep example environment files alongside the
Compose files.

**Lesson:** Configuration-as-code makes deployments easier to review and
rebuild. Document which Compose files are alternatives: this repository has
two Homepage definitions, and they should not both be launched unchanged on
the same machine.

## 3. Separating secrets from configuration

**Challenge:** Compose files need credentials, but committing real passwords
or tokens makes a public repository unsafe.

**Approach:** Reference environment variables in Compose and commit only
placeholder `.env.example` files. Ignore real `.env` files and credential
material.

**Lesson:** A placeholder file documents required settings without publishing
the values. If a credential has ever been exposed, rotate it; deleting it
from the latest version of a file is not enough if it remains in Git history.

## 4. Separating backups from source control

**Challenge:** VM images, photo libraries, and database directories can be
many gigabytes and may contain personal data.

**Approach:** Keep large backups and application data in private backup
storage. Track only the configuration and documentation needed to understand
or recreate the deployment.

**Lesson:** Git is for versioning source and configuration, not for storing
the live contents of every service. Maintain a separate backup and recovery
plan, and periodically test restores.

## 5. Treating monitoring as part of the system

**Challenge:** Self-hosted services can fail without an obvious indication
until someone tries to use them.

**Approach:** Include Uptime Kuma for availability monitoring and Homepage as
a central dashboard. Portainer provides a management interface for Docker.

**Lesson:** A dashboard helps with visibility, but it is not a substitute for
monitoring, logs, backups, or recovery procedures. After redeployment, verify
that monitoring checks actually detect a stopped service.

## Next validation steps

- Rebuild the Proxmox host and confirm the bridge/NAT configuration.
- Restore the Ubuntu VM and validate outbound connectivity.
- Deploy services from Compose using newly created secrets.
- Verify persistent volumes and test a backup restore.
- Confirm monitoring detects a deliberately stopped test service.
- Update this document with measured outcomes and any changes made.
