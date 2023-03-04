# lunchbox

_Setup for a compact OpenShift demo node aka lunchbox._

## One time router setup

1. Start up your GL-AR750S and plug in or connect to its wifi
2. Add the following to your `~/.ssh/config` to allow OpenWRT's only (insecure) crypto:

  ```
  Host 192.168.8.1
      PubkeyAcceptedAlgorithms +ssh-rsa
      HostkeyAlgorithms +ssh-rsa
    ```

3. `ssh root@192.168.8.1`
4. Effectively wildcard the entire `demo.local` domain to 192.168.8.10, which will
   be the SNO cluster's IP (we'll configure that later):

  ```
  uci add_list dhcp.@dnsmasq[0].address="/apps.sno.demo.local/192.168.8.10"
  uci add_list dhcp.@dnsmasq[0].address="/api.sno.demo.local/192.168.8.10"
  uci commit dhcp
  /etc/init.d/dnsmasq restart
  exit
  ```

## Prereqs

- The NVMe disk on the node must be empty. If not, LVM storage will not provision
    itself. If you can't format this disk before the install, you can do so after
    and let the LVM operator reconcile itself.

## Get started

Determine the MAC address of the network interface you'll use for the node. If
you don't know, just plug it in to the router, boot it up, and check the clients
list in the router's web GUI.

**Create a cluster using Assisted Installer at https://console.redhat.com**

1. Cluster details:
  a. Cluster name: `sno`
  a. Base domain: `demo.local`
  a. Install SNO
  a. Hosts' network configuration: Static IP
1. Network-wide configurations:
  a. DNS: `192.168.8.1`
  a. Machine network: `192.168.8.0` / `24`
  a. Default gateway: `192.168.8.1`
1. Host-specific configurations:
  a. MAC address: enter the MAC you determine for the node
  a. IP address: `192.168.8.10`
1. Operators: install OpenShift Virt and LVM Storage
1. Host discovery: Add host
  a. Provisioning type: full image file
  a. SSH public key: add your `~/.ssh/id_rsa.pub` here
  a. Generate Discovery ISO
1. Burn ISO to a flash drive, [Belena Etcher](https://www.balena.io/etcher) is easy

**Install the cluster**

1. Insert the flash drive and boot from it _one time_ (you can get a virtual console
    from the BMC web GUI, F11 launches the boot menu)
1. After a few minutes the node shows up in the Red Hat Console, click Next, Next,
    Next, and finally Install cluster
1. Wait until the installer is 100% complete, then copy the `kubeadmin` password
    displayed. Also download the `kubeconfig` file.
1. Navigate to https://console-openshift-console.apps.sno.demo.local/, accept
    the certificates, and log in with `kubeadmin` and the password you just copied.

**Smoke test**

1. Check that all operators are Succeeded
1. Check that the LVM Storage operator's `LVMCluster` is Ready.

**Bootstrap the cluster**

1. Apply bootstrap/10-gitops.yaml (either from the command line or the **+** in
    the web console). Wait for Upgrade status to show Up to date.
1. Apply bootstrap/20-application.yaml
