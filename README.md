# Qubes Proxy

This is a program to install a proxy network tool ([sing-box](https://sing-box.sagernet.org/)) in Qubes OS, designed to help Qubes OS users break through the blockade in a heavily censored network environment and give Qubes OS the ability to connect to the Tor network.

The project's main repository is on on [GitHub](https://github.com/glockmane/qubes-proxy).

## How it works

It provides a web service box based on the isolation mechanism of Qubes OS. It cleverly utilizes the global DNS IPs (10.139.1.1 and 10.139.1.2) of Qubes OS as the IPs of the tun devices and lets the traffic pass through it to provide a proxy network for other applications or service boxes.

## Usage scenarios

It can work in these scenarios and perhaps you have your own!

- sys-net <- sys-firewall <- **sys-proxy** <- AppVM(s)
- sys-net <- **sys-proxy** <- sys-firewall <- AppVM(s)

## Prerequisite

- Qubes OS
- Proxy Service Account

## Installation

Here create a proxy box, which is named `sys-proxy`, and then download the [sing-box](https://github.com/SagerNet/sing-box/releases) binary from GitHub.

```bash
[user@dom0 ~]$ qvm-create sys-proxy --class AppVM --label blue
[user@dom0 ~]$ qvm-prefs sys-proxy provides_network true
[user@dom0 ~]$ qvm-prefs sys-proxy autostart true
[user@dom0 ~]$ qvm-prefs sys-proxy memory 500
[user@dom0 ~]$ qvm-prefs sys-proxy maxmem 500
```

Next sing-box will be installed into the `/rw/usrlocal/bin` directory, and the configuration file `config.json` is installed into the `/rw/bind-dirs/etc/sing-box` directory.
The daemon runtime configuration file `sing-box.service` is installed into the `/rw/bind-dirs/etc/systemd/system` directory.

After completing the installation, you need to change `outbounds` in `/rw/bind-dirs/etc/sing-box/config.json` to your own proxy service.
Refer to [Sing-box Configuration](https://sing-box.sagernet.org/configuration/) for more configuration information.

```bash
[user@dom0 ~]$ qvm-start sys-proxy
[user@dom0 ~]$ qrexec-client -W -d sys-proxy user:'sh <(curl --proto "=https" -tlsv1.2 -SfL https://raw.githubusercontent.com/glockmane/qubes-proxy/refs/heads/main/install.sh)'
```

When you come to this step, you will have to restart the `sys-proxy` box.

```bash
[user@dom0 ~]$ qvm-shutdown --wait sys-proxy
[user@dom0 ~]$ qvm-start sys-proxy
```

Verify that the `sys-proxy` box confirms the operational status of the proxy service.

```bash
[user@dom0 ~]$ qrexec-client -W -d sys-proxy root:'journalctl -ft sing-box'
```

Finally, configure the application or service box's `netvm` to `sys-proxy`, and the following example configures `sys-whonix`'s network to `sys-proxy`, i.e., `sys-proxy` acts as the `sys-whonix` front proxy.

```bash
[user@dom0 ~]$ qvm-prefs sys-whonix netvm sys-proxy
```

## Related

- [Qubes OS](https://www.qubes-os.org/)
- [sing-box](https://sing-box.sagernet.org/)
- [sing-box GitHub](https://github.com/SagerNet/sing-box)
