# Generating Insecure Certificates for Development

<!--
  Copyright 2018-2021 Cargill Incorporated
  Licensed under Creative Commons Attribution 4.0 International License
  https://creativecommons.org/licenses/by/4.0/
-->

This document describes how to use the `splinter` CLI to generate certificates
for running the Splinter daemon in a development environment.

The Splinter daemon can be run with raw TCP connections and with TLS
connections. When developing against Splinter, you might want to run in TLS mode
without the effort of getting valid X.509 certificates from a certificate
authority. If so, you can use the `splinter` CLI to generate insecure
certificates and the associated keys.

The `splinter cert generate` command creates the following certificates and
keys:
  - `client.crt`
  - `client.key`
  - `server.crt`
  - `server.key`
  - `generated_ca.pem`
  - `generated_ca.key`

> **Note:** The `generated_ca` files will be used to create the development
> client and server certificates. Do not use these files as the trusted
> certificate authority.

This command writes the files to the default Splinter certificate directory,
`/etc/splinter/certs/`. To use a different directory (for example, if you
already have valid certificate files in this directory), you can change the
location by setting the `SPLINTER_CERT_DIR` environment variable or using the
`--tls-cert_dir` option on the command line.

## Prerequisites

This procedure requires a local development environment that includes the
`splinter` CLI and the Splinter daemon (`splinterd`).

## Procedure

1. Open a terminal window and navigate to the location of the `splinter` CLI
   on your local machine.

1. If you want to change the default certificate directory, set the
   `SPLINTER_CERT_DIR` environment variable. (Otherwise, you could
   use the `--cert-dir` option with `splinter` and `--tls-cert-dir` with
   `splinterd` commands in this procedure.)

1. Run the following command to generate certificates and keys.

   ``` console
   $ splinter cert generate
   ```

   The output shows the full path of each file.

   ``` console
   Writing file: /etc/splinter/certs/generated_ca.pem
   Writing file: /etc/splinter/certs/private/generated_ca.key
   Writing file: /etc/splinter/certs/client.crt
   Writing file: /etc/splinter/certs/private/client.key
   Writing file: /etc/splinter/certs/server.crt
   Writing file: /etc/splinter/certs/private/server.key
   ```

   Tip: To change the common name of the certificates, use the `--common-name`
   flag. Otherwise, the common name will default to `localhost`.

1. Start the Splinter daemon in TLS mode with the following `splinterd` command.

   The `--tls-insecure` flag turns off certificate authority validation (because
   the generated certificates are not signed by a common certificate authority).
   Without this flag, the Splinter daemon will get a TLS error when it tries to
   connect to another Splinter node.

   ``` console
   $ splinterd --node-id node-000 --tls-insecure --network-endpoints tcps://127.0.0.1:8044
   [2020-02-04 15:40:29.763] T["main"] WARN [splinterd] Starting TlsTransport in insecure mode
   ```

   * You can specify the generated certificate directory by using the
    `--tls-cert-dir` option, setting `SPLINTER_CERT_DIR` environment variable,
    or setting the location in the splinterd config, `splinterd.toml`. If
    necessary, you can specify each file separately with the command-line option
    or in the config file.

   * When the Splinter daemon starts, it logs a warning message that TLS
     has started in insecure mode. To verify that the correct certificates and
     keys were used, add `-vv` to increase the logging level.

   ``` console
   $ splinterd --node-id node-000 --tls-insecure -vv --network-endpoints tcps://127.0.0.1:8044
   .
   .
   [2021-11-10 11:24:40.483] T[main] WARN [splinterd::transport] Starting TlsTransport in insecure mode
   [2021-11-10 11:24:40.484] T[main] DEBUG [splinterd::transport] Using client certificate file: "/etc/splinter/certs/client.crt"
   [2021-11-10 11:24:40.484] T[main] DEBUG [splinterd::transport] Using client key file: "/etc/splinter/certs/private/client.key"
   [2021-11-10 11:24:40.484] T[main] DEBUG [splinterd::transport] Using server certificate file: "/etc/splinter/certs/server.crt"
   [2021-11-10 11:24:40.484] T[main] DEBUG [splinterd::transport] Using server key file: "/etc/splinter/certs/private/server.key"
   [2021-11-10 11:24:40.499] T[main] DEBUG [splinterd::config] Config: config_dir: /etc/splinter (source: Default)
   [2021-11-10 11:24:40.499] T[main] DEBUG [splinterd::config] Config: tls_ca_file: /etc/splinter/certs/ca.pem (source: Default)
   [2021-11-10 11:24:40.499] T[main] DEBUG [splinterd::config] Config: tls_cert_dir: /etc/splinter/certs (source: Default)
   [2021-11-10 11:24:40.499] T[main] DEBUG [splinterd::config] Config: tls_client_cert: /etc/splinter/certs/client.crt (source: Default)
   [2021-11-10 11:24:40.499] T[main] DEBUG [splinterd::config] Config: tls_client_key: /etc/splinter/certs/private/client.key (source: Default)
   [2021-11-10 11:24:40.499] T[main] DEBUG [splinterd::config] Config: tls_server_cert: /etc/splinter/certs/server.crt (source: Default)
   [2021-11-10 11:24:40.499] T[main] DEBUG [splinterd::config] Config: tls_server_key: /etc/splinter/certs/private/server.key (source: Default)
   [2021-11-10 11:24:40.499] T[main] DEBUG [splinterd::config] Config: network_endpoints: ["tcps://127.0.0.1:8044"] (source: CommandLine)
   [2021-11-10 11:24:40.499] T[main] DEBUG [splinterd::config] Config: advertised_endpoints: ["tcps://127.0.0.1:8044"] (source: Default)
   [2021-11-10 11:24:40.499] T[main] DEBUG [splinterd::config] Config: peers: [] (source: Default)
   [2021-11-10 11:24:40.499] T[main] DEBUG [splinterd::config] Config: peering_key: "splinterd" (source: Default)
   [2021-11-10 11:24:40.500] T[main] DEBUG [splinterd::config] Config: rest_api_endpoint: http://127.0.0.1:8080 (source: Default)
   [2021-11-10 11:24:40.500] T[main] DEBUG [splinterd::config] Config: registries: [] (source: Default)
   [2021-11-10 11:24:40.500] T[main] DEBUG [splinterd::config] Config: registry_auto_refresh: 600 (source: Default)
   [2021-11-10 11:24:40.500] T[main] DEBUG [splinterd::config] Config: registry_forced_refresh: 10 (source: Default)
   [2021-11-10 11:24:40.500] T[main] DEBUG [splinterd::config] Config: state_dir: /var/lib/splinter (source: Default)
   [2021-11-10 11:24:40.500] T[main] DEBUG [splinterd::config] Config: heartbeat: 30 (source: Default)
   [2021-11-10 11:24:40.500] T[main] DEBUG [splinterd::config] Config: admin_timeout: 30s (source: Default)
   [2021-11-10 11:24:40.500] T[main] DEBUG [splinterd::config] database: /var/lib/splinter/splinter_state.db (source: Default)
   [2021-11-10 11:24:40.500] T[main] DEBUG [splinterd::config] Config: tls_insecure: true (source: CommandLine)
   [2021-11-10 11:24:40.500] T[main] DEBUG [splinterd::config] Config: no_tls: false (source: Default)
   [2021-11-10 11:24:40.500] T[main] DEBUG [splinterd::config] Config: enable_biome_credentials: false (source: CommandLine)
   [2021-11-10 11:24:40.500] T[main] DEBUG [splinterd::config] Config: strict_ref_counts: false (source: Environment)
   [2021-11-10 11:24:40.534] T[main] DEBUG [splinterd::daemon] Listening for peer connections on ["tcps://127.0.0.1:8044"]
   [2021-11-10 11:24:40.548] T[main] INFO [splinterd::daemon] Starting SpinterNode with ID node-000
   .
   .
   ```

     With verbose logging, the Splinter daemon will log a debug message
     with the configuration used to start the daemon, including the location of
     the client and server certificate and keys.

## Troubleshooting

  If any certificate files already exist, `splinter cert generate` displays an
  error and stops. It does not create any new files.

  ``` console
  $ splinter cert generate

  Client certificate already exists: /etc/splinter/certs/client.crt
  Client key already exists: /etc/splinter/certs/private/client.key
  Server certificate already exists: /etc/splinter/certs/server.crt
  Server key already exists: /etc/splinter/certs/private/server.key
  CA certificate already exists: /etc/splinter/certs/generated_ca.pem
  CA key already exists: /etc/splinter/certs/private/generated_ca.key
  ERROR: action encountered an error: Refusing to overwrite files, exiting
  ```

  To create missing certificates and keys when some files already
  exist, add the `--skip` flag. The command will ignore the existing
  files and create any files that are missing.

  ``` console
  $ splinter cert generate --skip

  Client certificate exists, skipping: /etc/splinter/certs/client.crt
  Client key exists, skipping: /etc/splinter/certs/private/client.key
  Server certificate exists, skipping: /etc/splinter/certs/server.crt
  Server key exists, skipping: /etc/splinter/certs/private/server.key
  CA certificate exists, skipping: /etc/splinter/certs/generated_ca.pem
  CA key exists, skipping: /etc/splinter/certs/private/generated_ca.key
  ```

  To recreate the certificates and keys from scratch, use the  `--force` flag to
  overwrite all existing files.

  ``` console
  $ splinter cert generate --force

  Overwriting file: /private/etc/splinter/certs/generated_ca.pem
  Overwriting file: /private/etc/splinter/certs/private/generated_ca.key
  Overwriting file: /private/etc/splinter/certs/client.crt
  Overwriting file: /private/etc/splinter/certs/private/client.key
  Overwriting file: /private/etc/splinter/certs/server.crt
  Overwriting file: /private/etc/splinter/certs/private/server.key
  ```
