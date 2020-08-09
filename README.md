# Transparent Squid in a container

This is a trivial Dockerfile to build a proxy container.
It will use the famous Squid proxy, configured to work in transparent mode using jpetazzo's Squid in a Can.
Reconfigured to be quickly deployed for network usage rather than local caching.


## Why?

When dealing with tablet based devices, or difficulties in network troubleshooting while using complex firewall appliances, such as nested firewalls,
you may need to take advantage of caching updates, or temporarily bypassing user-based firewall restrictions and moving the device into the host vlan/IP-based authentication rule.


## How?


### Using Docker and iptables directly.

You can manually run these commands

```bash
docker run -p 3129:3129 -d jpetazzo/squid-in-a-can
```


### Using Compose



### Result




## What?

The `jpetazzo/squid-in-a-can` container runs a really basic Squid3 proxy.


## Tuning

The docker image can be tuned using environment variables.


### MAX_CACHE_OBJECT

Squid has a maximum object cache size. Often when caching debian packages vs
standard web content it is valuable to increase this size. Use the
`-e MAX_CACHE_OBJECT=1024` to set the max object size (in MB)


### DISK_CACHE_SIZE

The squid disk cache size can be tuned. use
`-e DISK_CACHE_SIZE=5000` to set the disk cache size (in MB)


### SQUID_DIRECTIVES_ONLY

The contents of squid.conf will only be what's defined in SQUID_DIRECTIVES
giving the user full control of squid.


### SQUID_DIRECTIVES

This will append any contents of the environment variable to squid.conf.
It is expected that you will use multi-line block quote for the contents.

Here is an example:

```bash
docker run -d \
    -e SQUID_DIRECTIVES="
    # hi ho hi ho
    # we're doing block I/O
    # hi ho hi ho
    " jpetazzo/squid-in-a-can
```


### Persistent Cache

Being docker when the instance exits the cached content immediately goes away
when the instance stops. To avoid this you can use a mounted volume. The cache
location is `/var/cache/squid3` so if you mount that as a volume you can get
persistent caching. Use `-v /home/user/persistent_squid_cache:/var/cache/squid3`
in your command line to enable persistent caching.

If you do that, make sure that the `persistent_squid_cache` directory is
writable by the right user. As I write these lines, the squid process
runs as user and group `proxy`, and their UID and GID both are 13; so
make sure that the directory is writable by UID 13, or by GID 13,
or (if you really can't make otherwise) world-writable (but please don't).

Note that if you're using Docker Mac, all volume I/O is handled by the
Docker Mac application, which runs as an ordinary process; so you won't
have to deal with permissions as long as you have read/write access to
a volume.


## Notes


### HTTPS support

It has been asked if this could support HTTPS. HTTPS is designed to prevent
man-in-the middle attacks, and a transparent proxy is effectively a MITM.
If you want to use squid for HTTPS proxying transparently you need to setup a
private CA certificate and push it to all your users so they trust the proxy.
An example of how to set this up can be found [here](http://roberts.bplaced.net/index.php/linux-guides/centos-6-guides/proxy-server/squid-transparent-proxy-http-https).

Without a CA certificate configured, the default behavior is to tunnel HTTPS
traffic using the `CONNECT` method. Squid makes the request on behalf of the
client but cannot decrypt or cache the requests or responses.


[CVE-2009-0801]: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-0801
