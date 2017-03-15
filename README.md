# Unseal

Unseal is a small, simple go binary that takes a yaml config file and unseals vault servers.

# Warning

This is currently a WIP, and is not considering production ready or in any way safe. The only way to use it is to store unseal keys in plain text.

Future versions will not have this requirements. Use at your own risk

# Why?

When initially deploying vault across multiple sites, you're probably deploying it in a HA config (ie with multiple vault servers in the cluster) and you'll need several people to unseal all of them to get started. This got quite annoying over multiple vault servers and multiple sites, so in order to speed it up, I wrote this little tool.

# Features

Some of the advantages you might gain over using the vault HTTP API or the standard vault binary

  - Zero touch interaction. Once you've written your yaml config, you can simply invoke the command and it'll unseal all the servers that need to be unsealed
  - Parallel execution. Each unseal command runs in a goroutine, meaning you can unseal multiple servers in a matter of seconds
  - Overwriting of unseal key stored in memory. The unseal key you use is zeroed out when the unseal operation is completed, meaning it can't be hijacked by malware etc (see considerations for more info)

# Usage

In order to use unseal, simply create a config file. Here's an example:


```yaml
# optional config containing a path
# to multiple CA certs to read
# while unsealing vaults
capath: "/path/to/ca/certs"
# An array of keys with vault config
hosts:
  - name: vault-server-1
    port: 8200
    key: <base64 encoded key>
  - name: vault-server-2
    port: 8200
    key: <base64 encoded key>
  - name: different-site-vault-server.example.com 
    port: 8200
    key: <different base64 encoded key>
```

The app will look for the config file in the following directories, in order:

 - `/etc/unseal/config.yaml`
 - `$HOME/.unseal/config.yaml`
 - `config.yaml` (in the directory you're running the binary from)

Once that's done, simply run the binary:

```bash
./unseal
Host: vault-server-1 unseal progress is now: 1 of 3
Host: vault-server-2 unseal progress is now: 1 of 3
Host: different-site-vault-server.example.com unseal progress is now: 1 of 3
```

Your vault server progress is now 1 of 3. Yay!

## Environment Variables

By default, vault will read some environment variables to do the unseal config. You can find them [here](https://www.vaultproject.io/docs/commands/environment.html)

You can use _some_ of these environment variables if you wish when using unseal.

 - `VAULT_CACERT`: Set this to the path of a CA Cert you wish to use to verify the vault connection. Note, this will use the same CA cert for all vaults
 - `VAULT_CAPATH`: An alternative to the above CA Path config option.
 - `VAULT_CLIENT_CERT`: An SSL client cert to use when connecting to your vaults. Note, this will use the same cert for all vaults
 - `VAULT_CLIENT_KEY`: An SSL client key to use when connecting to your vaults. Note, this will use the same key for all vaults
 - `VAULT_SKIP_VERIFY`: Skip SSL verification. This is not recommended in production use.

# Considerations

A few security considerations before you use this tool.

 - Your unseal key is clearly stored in plaintext in the yaml file. This is clearly a security issue. Please don't store your unseal key in plaintext permanantly.
 - While I've taken steps to overwrite the unseal key in memory, I am not a Golang expert and it may not be fool proof. If you think you can improve the implementation, pull requests will be warmly welcomed
 - I am just getting started with Golang, and therefore there may be errors, security issues and gremlins in this code. Again, pull requests are much appreciated.
 - There is currently no way of setting HTTPS certificates, so you must trust the certificate presented by the vault API


# Building

If you want to contribute, we use [glide](https://glide.sh/) for dependency management, so it should be as simple as:

 - cloning this repo into `$GOPATH/src/github.com/jaxxstorm/unseal`
 - run `glide install` from the directory
 - run `go build -o unseal main.go`
