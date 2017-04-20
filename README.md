# Docker RBAC & ABAC Authorization Plugin based on Casbin

This plugin controls the access to Docker commands based on authorization policy. The functionality of authorization is provided by [Casbin](https://github.com/hsluoyz/casbin-authz-plugin). Since Docker doesn't perform authentication by now, there's no user information when executing Docker commands. The access that Casbin plugin can control is actually what HTTP method can be performed on what URL path.

For example, when you run ``docker images`` command, the underlying request is really like:

```
/v1.27/images/json, GET
```

So Casbin plugin helps you decide whether ``GET`` can be performed on ``/v1.27/images/json`` base on the policy rules you write. The policy file is ``basic_policy.csv`` co-located with the plugin binary by default. And its content is:

```
p, /v1.27/images/json, GET
```

The above policy grants anyone to perform ``GET`` on ``/v1.27/images/json``.

For more information about the casbin.conf, Casbin model or more advanced Casbin policy usage, please refer to: https://github.com/hsluoyz/casbin

## Build

```bash
$ go get github.com/hsluoyz/casbin-authz-plugin
$ cd $GOPATH/src/github.com/hsluoyz/casbin-authz-plugin
$ make
$ sudo make install
```

## Run

### Run the plugin directly in a shell

```bash
$ cd /usr/lib/docker
$ ./casbin-authz-plugin
```

### Run the plugin as a systemd service

```bash
$ systemctl daemon-reload
$ systemctl enable casbin-authz-plugin
$ systemctl start casbin-authz-plugin
```

See whether the plugin starts correctly:

```bash
$ journalctl -xe -u casbin-authz-plugin -f
```

## Enable the authorization plugin on docker engine

### Step-1: Add authorization plugin to the docker engine configuration 

Please add the following cmdline flag to your docker engine (e.g. ExecStart line ``/lib/systemd/system/docker.service``)

```bash
--authorization-plugin casbin-authz-plugin
```

### Step-2: Restart docker engine

```bash
$ systemctl daemon-reload
$ systemctl restart docker
```

## Stop and uninstall the plugin as a systemd service

NOTE: Before doing below, remove the authorization-plugin configuration created above and restart the docker daemon.

Stop the plugin service:

```bash
$ systemctl stop img-authz-plugin
$ systemctl disable img-authz-plugin
```

Uninstall the plugin service:

```bash
$ make uninstall
```

## Contact

If you have any issues or feature requests, please feel free to contact me at:
- https://github.com/hsluoyz/casbin/issues
- hsluoyz@gmail.com (Yang Luo's email, if your issue needs to be kept private, please contact me via this mail)

## License

Apache 2.0