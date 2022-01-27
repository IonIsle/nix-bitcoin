# Nodeinfo
Run `nodeinfo` to see onion addresses and local addresses for enabled services.

# Managing services

NixOS uses the [systemd](https://wiki.archlinux.org/title/systemd) service manager.

Usage:
```shell
# Show service status
systemctl status bitcoind

# Show the last 100 log messages
journalctl -u bitcoind -n 100
# Show all log messages since the last system boot
journalctl -b -u bitcoind

# These commands require root permissions
systemctl stop bitcoind
systemctl start bitcoind
systemctl restart bitcoind

# Show the service definition
systemctl cat bitcoind
# Show all service parameters
systemctl show bitcoind
```

# Connect to RTL
Normally you would connect to RTL via SSH tunneling with a command like this

```
ssh -L 3000:localhost:3000 root@bitcoin-node
```

Or like this, if you are using `netns-isolation`

```
ssh -L 3000:169.254.1.29:3000 root@bitcoin-node
```

Otherwise, you can access it via Tor Browser at `http://<onion-address>`.
You can find the `<onion-address>` with command `nodeinfo`.
The default password location is `/secrets/rtl-password`.

# Connect to spark-wallet
### Requirements
* Android phone
* [Orbot](https://guardianproject.info/apps/orbot/) installed from [F-Droid](https://guardianproject.info/fdroid) (recommended) or [Google Play](https://play.google.com/store/apps/details?id=org.torproject.android&hl=en)
* [Spark-wallet](https://github.com/shesek/spark-wallet) installed from [direct download](https://github.com/shesek/spark-wallet/releases) or [Google Play](https://play.google.com/store/apps/details?id=com.spark.wallet)

1. Enable spark-wallet in `configuration.nix`

    Change
    ```
    # services.spark-wallet.enable = true;
    ```
    to
    ```
    services.spark-wallet.enable = true;
    ```

2. Deploy new `configuration.nix`

3. Enable Orbot VPN for spark-wallet

    ```
    Open Orbot app
    Turn on "VPN Mode"
    Select Gear icon under "Tor-Enabled Apps"
    Toggle checkbox under Spark icon
    ```

4. Get the onion address, access key and QR access code for the spark wallet android app

    ```
    journalctl -eu spark-wallet
    ```
    Note: The qr code might have issues scanning if you have a light terminal theme. Try setting it to dark or highlighting the entire output to invert the colors.

5. Connect to spark-wallet android app

    ```
    Server Settings
    Scan QR
    Done
    ```

# Connect to LND with Zeus
### Requirements
* Android phone
* [Orbot](https://guardianproject.info/apps/orbot/) installed from
  [F-Droid](https://guardianproject.info/fdroid) (recommended) or
  [Google Play](https://play.google.com/store/apps/details?id=org.torproject.android&hl=en)
* [Zeus](https://zeusln.app/) installed from
  [F-Droid](https://f-droid.org/en/packages/app.zeusln.zeus/) (recommended) or
  [Google Play](https://play.google.com/store/apps/details?id=app.zeusln.zeus)

1. Enable `restOnionService` in `configuration.nix`

    Change
    ```
    # services.lnd.restOnionService.enable = true;
    ```
    to
    ```
    services.lnd.restOnionService.enable = true;
    ```

2. Deploy new `configuration.nix`

3. Run command `lndconnect-rest-onion` (under `operator` user) to create a QR code for
   connecting to LND via the REST onion service.

4. Enable Orbot VPN for Zeus
    ```
    Open Orbot app
    Turn on "VPN Mode"
    Select Gear icon under "Tor-Enabled Apps"
    Toggle checkbox under Zeus icon
    ```

5. Scan the QR code with your Zeus wallet and start sending Satoshis privately

# Connect to electrs
### Requirements Android
* Android phone
* [Orbot](https://guardianproject.info/apps/orbot/) installed from [F-Droid](https://guardianproject.info/fdroid) (recommended) or [Google Play](https://play.google.com/store/apps/details?id=org.torproject.android&hl=en)
* [Electrum mobile app](https://electrum.org/#home) 4.0.1 and newer installed from [direct download](https://electrum.org/#download) or [Google Play](https://play.google.com/store/apps/details?id=org.electrum.electrum)

### Requirements Desktop
* [Tor](https://www.torproject.org/) installed from [source](https://www.torproject.org/docs/tor-doc-unix.html.en) or [repository](https://www.torproject.org/docs/debian.html.en)
* [Electrum](https://electrum.org/#download) installed

1. Enable electrs in `configuration.nix`

    Change
    ```
    # services.electrs.enable = true;
    ```
    to
    ```
    services.electrs.enable = true;
    ```

2. Deploy new `configuration.nix`

3. Get electrs onion address with format `<onion-address>:<port>`

    ```
    nodeinfo | jq -r .electrs.onion_address
    ```

4. Connect to electrs

    Make sure Tor is running on Desktop or as Orbot on Android.

    On Desktop
    ```
    electrum --oneserver -1 -s "<electrs onion address>:t" -p socks5:localhost:9050
    ```

    On Android
    ```
    Three dots in the upper-right-hand corner
    Network > Proxy mode: socks5, Host: 127.0.0.1, Port: 9050
    Network > Auto-connect: OFF
    Network > One-server mode: ON
    Network > Server: <electrs onion address>:t
    ```

# Connect to nix-bitcoin node through the SSH onion service
1. Get the SSH onion address (excluding the port suffix)

    ```
    ssh operator@bitcoin-node
    nodeinfo | jq -r .sshd.onion_address | sed 's/:.*//'
    ```

2. Create a SSH key

    ```
    ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
    ```

3. Place the ed25519 key's fingerprint in the `configuration.nix` `openssh.authorizedKeys.keys` field like so

    ```
    # FIXME: Add your SSH pubkey
    services.openssh.enable = true;
    users.users.root = {
      openssh.authorizedKeys.keys = [ "<contents of ~/.ssh/id_ed25519.pub>" ];
    };
    ```

4. Connect to your nix-bitcoin node's SSH onion service, forwarding a local port to the nix-bitcoin node's SSH server

    ```
    ssh -i ~/.ssh/id_ed25519 -L <random port of your choosing>:localhost:22 root@<SSH onion address>
    ```

5. Edit your deployment tool's configuration and change the node's address to `localhost` and the ssh port to `<random port of your choosing>`.
   If you use krops as described in the [installation tutorial](./install.md), set `target = "localhost:<random port of your choosing>";` in `krops/deploy.nix`.


6. After deploying the new configuration, it will connect through the SSH tunnel you established in step iv. This also allows you to do more complex SSH setups that some deployment tools don't support. An example would be authenticating with [Trezor's SSH agent](https://github.com/romanz/trezor-agent), which provides extra security.

# Initialize a Trezor for Bitcoin Core's Hardware Wallet Interface

1. Enable Trezor in `configuration.nix`

    Change
    ```
    # services.hardware-wallets.trezor = true;
    ```
    to
    ```
    services.hardware-wallets.trezor = true;
    ```

2. Deploy new `configuration.nix`

3. Check that your nix-bitcoin node recognizes your Trezor

    ```
    ssh operator@bitcoin-node
    lsusb
    ```
    Should show something relating to your Trezor

4. If your Trezor has outdated firmware or is not yet initialized: Start your Trezor in bootloader mode

    Trezor v1
    ```
    Plug in your Trezor with both buttons depressed
    ```

    Trezor v2
    ```
    Start swiping your finger across your Trezor's touchscreen and plug in the USB cable when your finger is halfway through
    ```

5. If your Trezor's firmware is outdated: Update your Trezor's firmware

    ```
    trezorctl firmware-update
    ```
    Follow the on-screen instructions

    **Caution: This command _will_ wipe your Trezor. If you already store Bitcoin on it, only do this with the recovery seed nearby.**

6. If your Trezor is not yet initialized: Set up your Trezor

    ```
    trezorctl reset-device -p
    ```
    Follow the on-screen instructions

7. Find your Trezor

    ```
    hwi enumerate
    hwi -t trezor -d <path from previous command> promptpin
    hwi -t trezor -d <path> sendpin <number positions for the PIN as displayed on your device's screen>
    hwi enumerate
    ```

8. Follow Bitcoin Core's instructions on [Using Bitcoin Core with Hardware Wallets](https://github.com/bitcoin-core/HWI/blob/master/docs/bitcoin-core-usage.md) to use your Trezor with `bitcoin-cli` on your nix-bitcoin node

# JoinMarket

## Diff to regular JoinMarket usage

For clarity reasons, nix-bitcoin renames all scripts to `jm-*` without `.py`, for
example `wallet-tool.py` becomes `jm-wallet-tool`. The rest of this section
details nix-bitcoin specific workflows for JoinMarket.

## Wallets

By default, a wallet is automatically generated at service startup.
It's stored at `/var/lib/joinmarket/wallets/wallet.jmdat`, and its mnmenoic recovery
seed phrase is stored at `/var/lib/joinmarket/jm-wallet-seed`.

A missing wallet file is automatically recreated if the seed file is still present.

If you want to manually initialize your wallet instead, follow these steps:

1. Enable JoinMarket in your node configuration

    ```
    services.joinmarket.enable = true;
    ```

2. Move the automatically generated `wallet.jmdat`

    ```console
    mv /var/lib/joinmarket/wallet.jmdat /var/lib/joinmarket/bak.jmdat
    ```

3. Generate wallet on your node

    ```console
    jm-wallet-tool generate
    ```
    Follow the on-screen instructions and write down your seed.

    In order to use nix-bitcoin's `joinmarket.yieldgenerator`, use the password
    from `/secrets/jm-wallet-password` and use the suggested default wallet name
    `wallet.jmdat`. If you want to use your own `jm-wallet-password`, simply
    replace the password string in your local secrets directory.

## Run the tumbler

The tumbler needs to be able to run in the background for a long time, use screen
to run it accross SSH sessions. You can also use tmux in the same fashion.

1. Add screen to your `environment.systemPackages`, for example

    ```
    environment.systemPackages = with pkgs; [
      vim
      screen
    ];
    ```

2. Start the screen session

    ```console
    screen -S "tumbler"
    ```

3. Start the tumbler

    Example: Tumbling into your wallet after buying from an exchange to improve privacy:

    ```console
    jm-tumbler wallet.jmdat <addr1> <addr2> <addr3>
    ```

    After tumbling your bitcoin end up in these three addresses. You can now
    spend them without the exchange collecting data on your purchases.

    Get more information [here](https://github.com/JoinMarket-Org/joinmarket-clientserver/blob/master/docs/tumblerguide.md)

4. Detach the screen session to leave the tumbler running in the background

    ```
    Ctrl-a d or Ctrl-a Ctrl-d
    ```

5. Re-attach to the screen session

    ```console
    screen -r tumbler
    ```

6. End screen session

    Type exit when tumbler is done

    ```console
    exit
    ```

## Run a "maker" or "yield generator"

The maker/yield generator in nix-bitcoin is implemented using a systemd service.

See [here](https://github.com/JoinMarket-Org/joinmarket-clientserver/blob/master/docs/YIELDGENERATOR.md) for more yield generator information.

1. Enable yield generator bot in your node configuration

    ```
    services.joinmarket.yieldgenerator = {
      enable = true;
      # Optional: Add custom parameters
      txfee = 200;
      cjfee_a = 300;
    };
    '';
    ```

2. Check service status

    ```console
    systemctl status joinmarket-yieldgenerator
    ```

3. Profit

# clightning

## Plugins

There are a number of [plugins](https://github.com/lightningd/plugins) available for clightning. Currently `nix-bitcoin` supports:

- helpme
- monitor
- prometheus
- rebalance
- summary
- zmq

You can activate and configure these plugins like so:

```nix
services.clightning = {
    enable = true;
    plugins = {
        prometheus.enable = true;
        prometheus.listen = "0.0.0.0:9900";
    };
};
```

Please have a look at the module for a plugin (e.g. [prometheus.nix](../modules/clightning-plugins/prometheus.nix)) to learn its configuration options.