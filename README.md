# IPSet Updater for Dynamic DNS Hosts

*IPSet Updater* is a Bash script designed to resolve hostnames with dynamic DNS and update IPs via `ipset` to whitelist in `iptables`. The net result is that you're effectively able to whitelist by hostname, which is quite helpful for clients with dynamic IPs. A good use case for this is locking down a Web server's SSH access by using `iptables` as a firewall, while having the ability to allow the legitimate owner of the server to access it via whitelist. By monitoring the result of resolving our dynamic DNS hostname(s), we get the IP(s) that must be whitelisted for access.

If you're not already using `iptables` as a firewall, and/or dynamic DNS, then you probably won't get much use out of this script.

> **Word of Caution**
>
> The script immediately flushes the `ipset` after resolving configured hostnames to keep it clean of stale entries. However, if you run this on any of your servers with a large amount of time between executions, you'll need to either run it more frequently (I use every 5 minutes in `crontab`) or manually check the whitelist regularly & remove stale entries for IPs from which you no longer connect.
>
> ***Note that connecting to your server from an untrusted/public network like a cafe or library while using this utility can create significant security risks. Use your brain.***

## Features

- Resolves hostnames with dynamic DNS
- Updates IPs using `ipset` for use in `iptables` (whitelisting, blacklisting, routing rules, etc.)
- Designed for use in `crontab`
- Requires root access

## Use Cases

If you've ever wanted to do the following in `iptables` but for hostnames with dynamic IPs, you've come to the right place.

- Whitelisting
- Blacklisting
- Routing rules

## Setup

1. Create your `ipset` & whitelist it in your `iptables` setup.
2. Adjust the configurable parameters as instructed below, add the `lynyx-ipset-updater` script to your `crontab` (e.g. every 5 min), and you're done!

### Example Config

Create your `ipset` as a `hash:ip` type revision 4. All of the following assumes root.

```bash
ipset create my-whitelist hash:ip family inet hashsize 1024 maxelem 65536
```

Assuming a restrictive firewall is already in place via `iptables`, add the following entry to whitelist your `ipset` for port 22 (SSH).

```text
-A INPUT -p tcp -m tcp --dport 22 -m set --match-set my-whitelist src -j ACCEPT
```

Set the environment variables while adding to .bashrc for persistence.

```bash
cat << EOF >> ~/.bashrc
> export LIU_IPSET_NAME="my-whitelist"
> export LIU_DOMAIN="dyndns.com"
> export LIU_SUB1="alpha"
> export LIU_SUB2="beta"
> # Add any more subdomains you need
> EOF
```

Add the script to crontab. This example runs every 5 minutes and redirects console output to a log file. Logging is optional but recommended.

```bash
*/5 * * * * lynyx-ipset-updater >> /var/log/lynyx-custom-tools/lynyx-ipset-updater.log 2>&1`
```

All done. Sit back and let the magic happen.

~[Relax & enjoy a cup of coffee.](https://images.unsplash.com/photo-1528962862197-29c4f24ccc04?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=2670&q=80)

## Configurable Parameters

```bash
read -r ipsetname domain <<<$(echo "${LIU_IPSET_NAME}" "${LIU_DOMAIN}") && read -a subdomains <<< "${LIU_SUB1} ${LIU_SUB2} ${LIU_SUB3}"
```

Set the environment variables with your desired values according to the guide below. Don't forget to add them to appropriate .*rc file so they persist for the user under whom you'll be running lynyx-ipset-updater. Keep in mind you typically need root privileges in order to use `ipset`.

| Variable                              | Description                                                                                                                             |
| ------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `LIU_IPSET_NAME`                      | Name of the `ipset` used specifically for your whitelisted hosts. Doesn't matter what it is but use a name you'll recognize.            |
| `LIU_DOMAIN`                          | The script is simple and assumes you'll connect from a single domain... but you can use as many subdomains as you like. See below.      |
| `LIU_SUB1` / `LIU_SUB2` / `LIU_SUB3`  | Set these to match *only the subdomain portion* of your whitelisted hostnames. If you don't need more than one, just delete the excess. |

## Logging

The script logs its activity. The log format is:

```text
[timestamp] | [logger] | [status] | [message]
```

## Functions

### Resolve Hosts

```bash
resolve_cmd() { dig +short $1; }
```

### Flush IP Set

```bash
ipsetflush_cmd() { ipset flush $ipsetname; }
```

### Add IP to IP Set

```bash
ipsetadd_cmd() { ipset add $ipsetname $1; }
```

### List IP Set

```bash
ipsetlist_cmd() { ipset list $ipsetname; }
```

## License

This project is licensed under the [MIT License](LICENSE).

## Acknowledgments

- EnzanNoMetsuke <3458546+EnzanNoMetsuke@users.noreply.github.com> - author of the original script
