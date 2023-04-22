# IPSet Updater for Dynamic DNS Hosts

IPSet Updater is a Bash script designed to resolve hostnames with dynamic DNS and update IPs via `ipset` to whitelist in `iptables`. The net result is that you're effectively able to whitelist by hostname, which is quite helpful for clients with dynamic IPs. A good use case for this is locking down a Web server's SSH access by using iptables as a firewall, while having the ability to allow the legitimate owner of the server to access it via whitelist. By monitoring the result of resolving a dynamic DNS hostname, we get the IP that needs to be whitelisted for access.

## Word of Caution

The script flushes the ipset immediately after resolving configured hostnames to keep it clean of stale entries; however, if you run this on any of your servers with a large amount of time between executions, you'll need to either run it more frequently (I use every 5 minutes) or manually check the whitelist regularly & remove stale entries for IPs from which you no longer connect. 

**Note that connecting to your server from a public network like a cafe or library while using this utility can create significant security risks.**

## Features

- Resolves hostnames with dynamic DNS
- Updates IPs using `ipset`
- Designed for use in `crontab`
- Requires root access

## Usage

This script is designed for use in crontab and requires root in order to access ipset.

## Configurable Parameters

```bash
read -r ipsetname domain <<<$(echo "IPSET-NAME-HERE" "DOMAIN-NAME-HERE") && read -a subdomains <<< "SUB1 SUB2 SUB3"
```
Replace the following variables with your own names:
`IPSET-NAME-HERE`           | Name of the ipset used specifically for your whitelisted hosts. Doesn't matter what it is but use a name you'll recognize.
`DOMAIN-NAME-HERE`          | The script is simple and assumes you'll connect from a single domain... but you can use as many subdomains as you like. See below.
`SUB1` / `SUB2` / `SUB3`    | Set these to match *only the subdomain portion* of your whitelisted hostnames. If you don't need more than one, just delete the excess.

## Logging

The script logs its activity. The log format is:

```
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