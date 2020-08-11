## Building

The following steps apply to Debian 10

```
apt-get install automake autoconf libssl-dev
./tools/setup
./configure --with-openssl=/usr/bin/openssl
make
```

## Deploying
The above steps will build the binaries. To use them on icinga, copy them to the right place:

```
cp plugins/check_http /usr/lib/nagios/plugins/check_http_2
```

And update the commands configuration at `/etc/icinga2/conf.d/commands.conf`, referencing your new binary:

```
// platform-team alternative http check
object CheckCommand "http2" {
  import "ipv4-or-ipv6"

  command = [ PluginDir + "/check_http_2" ]

  arguments = {
    "-H" = {
      value = "$http_vhost$"
      description = "Host name argument for servers using host headers (virtual host)"
    }
...
```

Then use it in `/etc/icinga2/conf.d/services.conf`:

```
apply Service for (http_vhost => config in host.vars.http_vhosts) {
  import "generic-service"
  // check_command = "http"
  check_command = "http2"
  check_interval = 5
  retry_interval = 1
  max_check_attempts = 10

  vars.http_address = host.address
  vars.http_vhost = host.address
  vars.http_ssl_force_tlsv1_or_higher = true
  vars.http_warn_time = 15
  vars.http_critical_time = 60
  vars.http_onredirect = "follow"
  vars.http_sni = true
  vars.http_timeout = 180
  vars.http_string_warning = "down for maintenance"
  vars += config
}
```

Finally reload icinga2:
```
icinga2 daemon -C
systemctl reload icinga2
```

