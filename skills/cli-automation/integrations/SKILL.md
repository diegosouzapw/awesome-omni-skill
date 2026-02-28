---
name: integrations
description:
  NextDNS integration guides for third-party platforms and services. This skill should be used when
  integrating NextDNS with routers, home automation systems, network management platforms, or other
  third-party services. Triggers on tasks involving Tailscale, Home Assistant, Ubiquiti, pfSense,
  OpenWrt, or other platform-specific NextDNS configurations.
license: MIT
metadata:
  author: tuanductran
  version: '1.0.0'
---

# NextDNS integration skills

## Capability rules

| Rule                                                                       | Keywords                                                                                                                     | Description                                                                                                                      |
| -------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| [Browser Native DoH Configuration](rules/browsers.md)                      | browser, chrome, edge, firefox, doh, dns-over-https, secure-dns, encrypted-dns, corporate, proxy                             | Configure DNS-over-HTTPS directly in browsers for encrypted DNS without system-wide changes or admin privileges                  |
| [DNSMasq Integration](rules/dnsmasq-integration.md)                        | dnsmasq, dns, router, client reporting, conditional configuration, port configuration, setup-router                          | Configure DNSMasq and NextDNS to run together while maintaining client reporting and conditional configuration features          |
| [Mobile Native Encrypted DNS Configuration](rules/mobile-native.md)        | mobile, android, ios, iphone, ipad, private-dns, dot, dns-over-tls, configuration-profile, mobileconfig, native, lightweight | Configure native encrypted DNS on Android and iOS devices without battery-draining background apps                               |
| [OpenWrt Integration](rules/openwrt.md)                                    | openwrt, router, installation, upgrade, troubleshooting, ssh, luci                                                           | Installation, upgrade, and troubleshooting guidance for NextDNS on OpenWrt routers                                               |
| [Home Assistant Integration](rules/home-assistant.md)                      | home assistant, hass, integration, automation, parental control, api, sensors, switches, blocklist                           | Enable automated DNS control and monitoring through Home Assistant smart home platform with time-based rules and analytics       |
| [Tailscale Integration](rules/tailscale.md)                                | tailscale, vpn, mesh network, doh, dns-over-https, acl, nodeattrs, global nameserver, split dns                              | Configure NextDNS with Tailscale mesh VPN using DoH, global nameservers, and per-device profile assignment via ACLs              |
| [Ubiquiti (UniFi) Integration](rules/ubiquiti.md)                          | ubiquiti, unifi, udm, uxg, dream machine, gateway, content filtering, ad blocking, ssh, cli, dns shield                      | Deploy NextDNS on Ubiquiti UniFi devices using DNS Shield or CLI, with critical guidance on avoiding built-in DNS conflicts      |
| [pfSense and OPNsense Integration](rules/pfsense-opnsense.md)              | pfsense, opnsense, firewall, router, unbound, dns-over-tls, dot, freebsd, dns resolver                                       | Deploy NextDNS on pfSense and OPNsense firewalls with proper encrypted DNS configuration and platform-specific guidance          |
| [Public DNS and AdGuard Home Integration](rules/public-dns-and-adguard.md) | public dns, adguard, anycast, doh, dot, browser setup, windows, android, ios, upstream dns, bootstrap dns                    | Configure NextDNS public DNS servers on browsers and operating systems, and integrate with AdGuard Home as upstream DNS provider |
| [Synology Integration (DSM and SRM)](rules/synology.md)                    | synology, dsm, srm, nas, router, ssh, cli, dhcp, network                                                                     | Install NextDNS on Synology NAS (DSM) and routers (SRM) with proper SSH access and network configuration                         |
| [AsusWRT-Merlin](rules/asuswrt-merlin.md)                                  | asus, merlin, router, ssh, installation, setup                                                                               | Deploy NextDNS on Asus routers with Merlin firmware                                                                              |
| [EdgeRouter and Firewalla](rules/edgerouter-firewalla.md)                  | edgerouter, edgeos, firewalla, gateway, ssh, professional                                                                    | Deploy NextDNS on EdgeRouter and Firewalla hardware                                                                              |
| [MikroTik DoH Setup](rules/mikrotik-setup.md)                              | mikrotik, routeros, doh, dns-over-https, networking                                                                          | Configure NextDNS via DNS-over-HTTPS on MikroTik RouterOS devices                                                                |
| [Kubernetes Integration](rules/kubernetes.md)                              | kubernetes, k8s, coredns, daemonset, dns policy, container                                                                   | Deploy NextDNS CLI as a node-level DNS proxy in Kubernetes clusters                                                              |

## Efficiency rules

| Rule                                            | Keywords                                      | Description                            |
| ----------------------------------------------- | --------------------------------------------- | -------------------------------------- |
| [Backup and Failover](rules/backup-failover.md) | backup dns, failover, redundancy, reliability | Ensure connectivity during DNS outages |

## Adding new integration rules

When adding a new integration rule:

1. Create a new file in `rules/` using kebab-case naming
2. Follow the rule template structure from `templates/rule-template.md`
3. Include platform-specific configuration examples
4. Add troubleshooting steps for common issues
5. Register the rule in the appropriate table above
6. Run `pnpm lint:fix` to ensure compliance

### Example integration platforms

Consider creating rules for these popular platforms:

**Capability Rules**:

- Tailscale (mesh VPN)
- Home Assistant (smart home automation)
- Ubiquiti UniFi (network controller)
- OpenWrt (router firmware)
- pfSense (firewall/router)
- Docker (containerization)
- Kubernetes (orchestration)
- Synology NAS (network storage)

**Efficiency Rules**:

- Multi-platform deployment strategy
- Integration testing and validation
- Backup DNS and failover configuration
