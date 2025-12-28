# XTLS

## Configuration

There are two ways to configure the setup. You can do it [manually](#Manual-configuation), but you still are going to need to use `./config-ctl gen` to generate certificates. You also can use [config-ctl](#Configuration-with-config-ctl) script, but you still need to configure your static content by hands.

### Manual configuration

Fill `inventory.ini` with server configuration:

```ini
[servers]
<your-domain-name>
```

It must be a real domain name, and your SSH needs to be configured so that you can connect to the server by simply typing `ssh <your-domain-name>`.

Set connection password in `secrets.yml`:

```yaml
xray_server:
  path: <http-path-to-xray>
xray_clients:
  - email: user1@example.com
    id: <user-uuid>
```

Emails exist so you can distinguish users not only by their UUIDs. They don't have to be real and may contain arbitrary strings.

To generate ids you can use `uuidgen -r`. The same goes for `xray_server.path` parameter.

Then you need to configure static.

```sh
$ mkdir static
$ cat > static/index.html <<EOF
<DOCTYPE! html>
<html>
  <head><title>Welcome!</title></head>
  <body><p>Nothing suspicious here!</p></body>
</html>
EOF
```

It expects you to create an `index.html` file, but you may add any data you need in the directory.

## Configuration with config-ctl

`./config-ctl` script provides basic configuration management features. It's concerned only with local configuration which needs to by applied separately by [deploy.yml](#Deploy) playbook.

When the setup is not configured, you need to initialize it with

```sh
$ ./config-ctl init <your-domain-name>
```

After the initialization (either manual or scripted) you can use `add` and `remove` commands to manage users:

```sh
$ ./config-ctl add <user-email>
```

To list all the saved users, run

```sh
$ ./config-ctl list
```

To remove an existing user, run

```sh
$ ./config-ctl remove <email-or-id>
```

Validation of the configuration may be attempted with

```sh
$ ./config-ctl validate
```

Script documentation is available via

```sh
$ ./config-ctl help
```

or if you run the script without arguments.

## Deploy

```sh
$ ansible-playbook deploy.yml
```

The script does the following.
1. Sets up and configures xray.
2. Installs acme.sh, issues a TLS certificate for your domain and sets up its renewal.
3. Sets up nginx as a reverse proxy for the xray and to serve static.
4. Blocks all the connections to the server except from 22 (ssh), 80 (http), and 443 (https) ports.
5. Starts xray with the configuration provided.

To apply changes in an already deployed configuration, you can run

```sh
$ ansible-playbook deploy.yml --tags reconfigure
```

This way you skip all the installation steps.

You can also redeploy static with the following command

```sh
$ ansible-playbook deploy.yml --tags static
```

## Cleanup

You can remove most of the side-effects of the script by running

```sh
$ ansible-playbook shutdown.yml
```

You also can just stop the service without removing all the configuration files and executables by running

```sh
$ ansible-playbook shutdown.yml --tags stop
```

## Usage

You should be able to use any [Xray client](https://github.com/XTLS/Xray-core?tab=readme-ov-file#gui-clients) you like.

### Linux

[Xray-core](https://github.com/XTLS/Xray-core) provides a binary that can be run with

```sh
$ xray -config <your-config-file>
```

### Windows

### Android

Install [Xray](https://github.com/SaeedDev94/Xray) and import generated config.

### iOS

Install [Streisand App](https://apps.apple.com/us/app/streisand/id6450534064) and import generated config.
