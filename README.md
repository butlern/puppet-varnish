# Varnish

Puppet module for varnish.

### Example usage

Include with default parameters:
```puppet
varnish::instance { 'instance0' : }
```

Include with multiple different storage types:
```puppet
varnish::instance { 'instance0' :
  storage_file => ['file,/mnt/varnish/instance0/varnish_storage.bin,70%',
                   'persistent,/data/varnish/varnish_storage.bin,20%']
}
```

The varnish::instance type is fully configurable. All of the varnish
options are available as parameters. If, for instance, you want to
increase the thread pool max size you could specify that as a param.

All of the parameters are set to the defaults as defined in varnishd(1)

```puppet
varnish::instance { 'instance0' :
  thread_pool_max => '1500',
}
```

### VMODS

The varnish::instance class supports installing vmods. Currently it expects
them to be installed via APT. There are plans to support building it from
source, but that would require a custom provider which is not yet built.

To ensure that a given varnish::instance has the throttle vmod installed
that it needs:

```puppet
varnish::instance { 'instance0' :
  vmods => ['libvmod-throttle'],
}
```

### Configs

The varnish::instance type ships with a pretty simple config. You can pass
in your own vcl config by just specifying the following:

```puppet
varnish::instance { 'instance0' :
  vcl_conf => 'puppet:///path/to/config/file.vcl',
}
```

Or if your config is a template, you can do that as well:

```puppet
varnish::instance { 'instance0' :
  vcl_conf => 'path/to/template/file.vcl.erb',
}
```

**Note that if you pass in your own config, you'll have to handle the backend
configuration, backend probes, vmod imports, acls yourself**

#### Extra conf

There is an extra_conf parameter that you can use as a way to customize the
default vcl conf further, but also retaining much of the scaffolding present
in the default vcl conf. When multiple subroutines are defined in varnish,
varnish will concatentate them together, for example:

```
vcl_recv {
    if (req.restarts == 0) {
        if (req.http.x-forwarded-for) {
            set req.http.X-Forwarded-For =
                req.http.X-Forwarded-For + ", " + client.ip;
        } else {
            set req.http.X-Forwarded-For = client.ip;
        }
    }
}

vcl_recv {
    # Serve up stale data for this long
    set req.grace = 6s;
}
```

Will result in the following compiled version:

```
vcl_recv {
    if (req.restarts == 0) {
        if (req.http.x-forwarded-for) {
            set req.http.X-Forwarded-For =
                req.http.X-Forwarded-For + ", " + client.ip;
        } else {
            set req.http.X-Forwarded-For = client.ip;
        }
    }
    # Serve up stale data for this long
    set req.grace = 6s;
}
```

As long as a return(SUBROUTINE) is not hit somewhere higher up, processing
will continue on through the vcl. The default vcl conf that ships with this
module has no return statements (except for the PURGE request) so you can
safely **append** your custom vcl configurations by just passing them in
as extra_conf.

```puppet
varnish::instance { 'instance0' :
  extra_conf => 'puppet:///path/to/extra/conf.vcl'
}
```

### Testing

See [LICENSE](LICENSE) file.
