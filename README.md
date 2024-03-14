# HowTo Install tailscale on OpenGear
Short write-up on how to install tailscale on OpenGear.

This is currently tested and used on the OpenGear Models `ACM7004-5-L` and `IM7216-2` with the OpenGear up to firmware Version `4.13.6` and tailscale version `1.62`.

There MAY be some adjustments needed if deploying older tailscale binaries.

## Prerequirements
- SSH access to the OpenGear
- Internet Access for the OpenGear
- Access Tailscale AdminConsole (Auth keys)

## General
Connecting the OpenGear into the tailscales tailnet requires __either__ a tailscales Auth keys __or__ a manual login after installing the tailscale. We will focus on the __Auth keys__ as this does not require manual intervention and can be automated in ZTP environment.

# Tailscale
## Creating the Auth key
- Login into the tailscales AdminConsole
- Goto `Settings / Personal Settings / Keys` 
    - Click `Generate auth key...` just below __Auth keys__ as desired
      (For Mass-Deployment you MAY want to tick the `Reuseable` Option)
    - Remember the `tskey-auth-XXX`

Additionally you need the tailscale binaries to install on the OpenGear box. In our Sample we're using the `ACM7004-5-L` on an `arm` platform. Check `cat /proc/cpuinfo` on the OpenGear if unsure about the used architecture.

You may download and scp the binaries or use the download-urls as we will use in our example install: https://pkgs.tailscale.com/stable/#static

# OpenGear

Connect to the OpenGear CLI through SSH and perform the following steps.

Depending on the used model, you may check and adjust your install path via `mount` or `df`.
For the IM7216 we're using `/var/mnt/storage.usb` and for the `ACM7004-5-L` we're using `/var/mnt/storage.nvlog`.

## 1) Sample Install of tailscale to the OpenGear (manual)

In this example we're using a `ACM7004-5-L`. 
To keep it simple: It's assumed that you have direct SSH Access to the OpenGear and that your local working directory is in the cloned repository folder.

1. Create the tailscale directory and copy the tailscale/tailscaled binaries into

    ```
    # Execute on OpenGear CLI:
    TAILSCALE_INSTALL_DIR="/var/mnt/storage.nvlog/tailscale"
    TAILSCALE_URL="https://pkgs.tailscale.com/stable/tailscale_1.62.0_arm.tgz" #ADJUST
    mkdir -p ${TAILSCALE_INSTALL_DIR}
    mkdir -p /etc/config/scripts
    curl -o /tmp/ts.tar.gz ${TAILSCALE_URL}
    tar xzvf /tmp/ts.tar.gz -C /tmp/
    cp /tmp/tailscale_*/tailscale* ${TAILSCALE_INSTALL_DIR}/
    ```
2. Copy the init-script and the logrotate file
   ***Make sure that you adjust the AUTHKEY (and if needed DIR) variable of the `tailscale-init-script` before _or_ after the copy!***
    ```
    # Execute locally
    scp samples/tailscale-init-script root@<OPENGEAR_IP>:/etc/config/scripts/tailscale
    scp samples/99-tailscale.conf root@<OPENGEAR_IP>:/etc/config/logrotate.d/
    ```
3. Make the init-script executable and add it to rc.local.
    ```
    # Execute on OpenGear CLI
    chmod +x /etc/config/scripts/tailscale
    echo '/etc/config/scripts/tailscale restart&' >> /etc/config/rc.local
    ```
4. Reboot the OpenGear and wait for it popping up in tailnet.
    ```
    # Execute on OpenGear CLI
    reboot
    ```
## 2) Sample Install of tailscale to the OpenGear (ztp/dhcp)
TBD

### Debug
- `tailscale status` can be executed 
    - via `/etc/config/scripts/tailscale status`
    - via `/var/mnt/storage.nvlog/tailscale/tailscale status`
- tailscale log is located in `/var/log/tailscaled.log`
- tailscale is not coming up after reboot
    - Check that your `tskey-auth` in `/etc/config/scripts/tailscale` is valid and not expired
    

### Factory-Reset OpenGear and remove tailscale
While logged into the OpenGear via SSH:
1. delete the tailscale directory and files within
    `rm -rf /var/mnt/storage.nvlog/tailscale/`
2. factory-reset the OpenGear
    `flatfsd -i`
    
