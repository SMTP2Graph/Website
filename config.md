# Config

## App registration

You will always need an Application Registration in Entra ID (Azure AD) to be able to send messages over the Exchange Online platform.
Instructions on how to create and configure this can be found [here](app-registration.md).

## Minimal

This is the minimal config, which will start the SMTP server on port 25 and relay any submitted message over the application registration.

```yaml
mode: full

send:
    appReg:
        tenant: contoso
        id: 01234567-89ab-cdef-0123-456789abcdef
        secret: VGhpcyBpcyB2ZXJ5IHNlY3JldCE=
```

## Configuration validation

There's a schema for the config that will provider additional information, add autocomplete and detect issues while editing the config file. You can load this schema from:
`https://raw.githubusercontent.com/SMTP2Graph/SMTP2Graph/main/config.schema.json`

<details>
<summary>VSCode usage</summary>

1. Make sure you have the [YAML extension](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) installed
2. Add this line to the top of you `config.yaml`: `# yaml-language-server: $schema=https://raw.githubusercontent.com/SMTP2Graph/SMTP2Graph/main/config.schema.json`

</details>

## Alternative config file/path

You can supply an alternative config file/path when starting SMTP2Graph by supplying the `--config` argument. You startup commend would look like: `smtp2graph-win-x64.exe --config=myconfig.yaml`


## Config options
A full example config file can be found here: [https://github.com/SMTP2Graph/SMTP2Graph/blob/main/config.example.yml](https://github.com/SMTP2Graph/SMTP2Graph/blob/main/config.example.yml)

### Mode
There are 3 modes to chose from: `full`, `receive` and `send`.

**Full**:  
You would normally want to use this mode. It will accept messages on it's SMTP server and relay them over Exchange Online.

**Receive**:  
This will queue all received messages in EML/RFC822 format (by default in the folder `mailroot/queue`).

**Send**:  
This will only pickup EML/RFC822 files from the queue. SMTP2Graph will not be running the SMTP server.

!> Do not run multiple `send`/`full` instances for the same queue folder, this can cause conflicts



### Send options

#### Send: Application Registration (required)

An Entra ID (Azure AD) Application Registration is needed to give SMTP2Graph access to relay messages over your Exchange Online tenant.  
Instruction on how to create/configure an application registration can be found [here](app-registration.md).

Example:
```yaml
send:
  appReg:
    tenant: contoso # This should be your tenant name (what comes before .onmicrosoft.com)
    id: 01234567-89ab-cdef-0123-456789abcdef # You application ID
    certificate: # The certificate you've registered with the app registration
      thumbprint: 0123456789ABCDEF0123456789ABCDEF01234567
      privateKeyPath: client.key
```
?> SMTP2Graph also supports an application secret by using the property `secret` instead of `certificate`, but it's recommended to use a certificate

#### Send: Retry limit

When for some reason SMTP2Graph fails to send the message (a connection issue for example), this option defined the number of retries before failing (failed messages will be placed in `mailroot/failed`).  
Default: `3`

Example:
```yaml
send:
  retryLimit: 10 # Fail after 10 retries
```

#### Send: Retry interval

This option defines the number of minutes to wait between [the retries](#send-retry-limit).  
Default: `5`

Example:
```yaml
send:
  retryInterval: 10 # Wait 10 minutes between retries
```

#### Send: Force mailbox

SMTP2Graph always sends from a mailbox (user or shared), the send message will appear in the `Sent items` folder of this mailbox.  
By default it sends from the mailbox supplied as the from address. If the app registration doesn't have permissions to send from this mailbox it will fail to relay the message.

?> See [Mail from distribution list](app-registration.md#mail-from-distribution-list) for more information

Example:
```yaml
send:
  forceMailbox: smtp2graph@example.com
```


### Receive options

#### Receive: port

This defines the port on which SMTP2Graph accepts SMTP connections.  
Default: `25`

Example:
```yaml
receive:
  port: 587 # Listen on port 587
```

#### Receive: listen address

Only accept SMTP connection on a specific (local) IP address.  
Default: all available addresses

Example:
```yaml
receive:
  listenAddress: 127.0.0.1 # This makes SMTP2Graph only accept SMTP connection from the local host
```

#### Receive: secure

Require a secure (TLS) connection.  
With this option enabled clients can only create a secure connection, with `secure: false` clients can still upgrade to a secure connection after connecting.  
It's strongly recommended to also define `tlsKeyPath` and `tlsCertPath` with this option.  
Default: `false`

```yaml
receive:
  secure: true
  port: 465 # Port 465 is normally used for secure SMTP
  tlsKeyPath: cert.key # Path to your private key
  tlsCertPath: cert.crt # Path to your certificate
```

#### Receive: max size

The maximum accepted message size. This option accepts a number ending with 'k' (Kilobytes) or 'm' (megabytes).  
Default: `100m`

Example:
```yaml
receive:
  maxSize: 25m # Accept message up to 25MB
```

#### Receive: banner

The welcome banner that's shown when a client connects to the SMTP server.  
Default: `SMTP2Graph <versionnumber>`

Example:
```yaml
receive:
  banner: My SMTP server
```

#### Receive: IP whitelist

Only accept messages for clients coming from the listed IP addresses or subnets.  
Default: accept any

Example:
```yaml
receive:
  ipWhitelist:
    - 127.0.0.1 # Accept from local host
    - 192.168.1.0/24 # Accept from subnet 192.168.1.0 - 192.168.1.255
    - ::1 # Accept from IPv6 local host
    - fe80::/10 # Accept from any link-local address
```

#### Receive: allowed from

Only accept messages using these from email addresses.  
It's recommended to use this option to make sure the SMTP server only accepts messages that it's able to relay.  
Default: accept any

Example:
```yaml
receive:
  allowedFrom:
    - noreply@example.com
    - something@example.com
```

#### Receive: users

Require clients to authenticate before being able to send messages.

Example:
```yaml
receive:
  requireAuth: true # Without this option clients can still continue without authenticating
  allowInsecureAuth: true # Allow clients to authenticate without upgrading to a secure (TLS) connection
  users:
    - username: testuser1
      password: P@ssword!
      allowedFrom: # Optional
        - testuser1@example.com
    - username: testuser2
      password: P@ssword!
      allowedFrom: # Optional
        - testuser2@example.com
```

#### Receive: rate limit

Stop accepting connection after a certain amount of connection within a certain time period.  
Default: `duration: 600` (600 seconds = 10 minutes), `limit: 10000`

Example:
```yaml
receive:
  rateLimit:
    duration: 60 # Time period in seconds (so this is 1 minute)
    limit: 10 # Accepted number of connection in the time period
```

#### Receive: auth limit

Brute force protection. Block login attempts after a certain number of failed attempts in a certain time period.  
Default: `duration: 60` (60 seconds = 1 minute), `limit 10`

Example:
```yaml
receive:
  duration: 30 # Time in seconds (so this is half a minute)
  limit: 5 # Maximum number of attempts in the time period
```
