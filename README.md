# NGINX Work Distributor

This project was created to be a "jumping-off point"
to have `NGINX` configured as a load-balancer and
reverse-proxy for a number of micro-services under
high load. Don't take my work as gospel, read the
docs and make adjustments based on your needs.

Also, note that you will need to tune your Linux configurations separately. There are several
great posts on this topic floating around the
internet including [this one from the NGINX
blog](https://www.nginx.com/blog/tuning-nginx/)
which also covers a lot of the configurations in
this repository.

## Setup

The config files in this project assume that the
default `NGINX` file structure is left in-tact and
that this repo is cloned into a sub-directory of the
nginx/conf directory named `distributor`. It is also
assumed that your main `nginx.conf` is empty except
for an include to the `distributor/global.conf`
file:

```sh
cd /path/to/your/nginx.conf/directory
git clone git@github.com:dgarlitt/nginx-work-distributor.git distributor
mv nginx.conf nginx.conf.original
echo "include distributor/global.conf;" > nginx.conf
```

## Logging
In `global.conf` the `access_log` is turned off,
which disables the writing of access logs for all
routes. This was done to reduce disk I/O from
potentially superfluous logging.

### Alternative Access Log Configurations
Access logs may be an important source of
information for you and, when enabled, can be tuned
to perform better. Listed below are some additional options for getting better performance from the
`access_log` directive.
[See the NGINX docs for more info](http://nginx.org/en/docs/http/ngx_http_log_module.html#access_log)


#### Buffer and Flush
The access log can be configured to only write when the buffer is full or the flush timeout is reached. By default, NGINX doesn't buffer writes to the
access log.

```sh
access_log /path/to/log.file buffer=128K flush=30s;
```

In the above case, the access log will be written to when the 128K buffer fills or when the flush timeout is reached, whichever comes first. In place of `buffer`, `gzip=[compression-level]` can be specified which will automatically buffer.

#### Conditional Logging
Logging could be adjusted to use a condition for
more granular control over what is logged by
following the the advice of this
[stackoverflow post](http://stackoverflow.com/questions/17423173/disable-logging-in-nginx-for-specific-request).

#### Disabled Per-Server
Alternatively, it could also be changed so that
the access log is enabled by default in the
`global.config`:

```sh
access_log /some/file/path/to/some.log
```

and then turned off using `access_log off` on a
per-server basis inside of each `server` block:

```sh
server {
  access_log off
  more server configs...
}
```

Note, however, that this would be useless in the
default configuration for this repo, as there is
only one `server` block which is located in the
`route.config` file.

### Send Logs to Syslog

Additionally, access or error log output can be
directed to syslog or a
unix-domain socket, [see here for more info](https://www.nginx.com/resources/admin-guide/logging-and-monitoring/).

### Log Rotation and More...

DigitalOcean posted
[this great article](https://www.digitalocean.com/community/tutorials/how-to-configure-logging-and-log-rotation-in-nginx-on-an-ubuntu-vps)
with more logging options including how to configure
log-rotation in `NGINX`.

## Keepalive

Make an effort to tune the keepalive settings to
make sense for your stack. Moreover, optimize the
`keepalive` property on a per-upstream basis.
[More info from NGINX.](https://www.nginx.com/blog/http-keepalives-and-web-performance/)

## Disclaimer

Please be mindful that the files in this repo make
use of some advanced configuration options available
in `NGINX`. As such, before using this in your own
projects, it would be wise to go through the files
and make sure you understand each of the settings
and adjust them according to your needs. I cannot
guarantee that these settings will work for
everyone, so use at your own risk. Other than that,
best of luck!

## Contributing

Contributions are always welcome. If you would like to contribute to this project, follow these easy steps:

 - Fork this repository
 - Create a feature branch for your changes
 - If you have a lot of commits, rebase from master
 and squash your commits to a single commit with a
 meaningful message
 - Make sure that your configurations are passing

 ```sh
 sudo nginx -t
 ```

 - Make sure your commit messages follow [this guide](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
 - Open a pull request
