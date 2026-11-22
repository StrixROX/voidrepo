# voidrepo
Simple bash script to initialise and update a your custom void repository using [Void container image](https://github.com/void-linux/void-containers/pkgs/container/void-glibc) via [Podman](https://docs.podman.io/en/latest/) (rootless).

# Pre-requisites

Install `podman` on your host machine as per your distro.

# Usage

1. Clone this repository

```bash
git clone https://github.com/StrixROX/voidrepo.git myvoidrepo
```

2. Add the name of packages you want to build in `packages.list`.

Example:
```
genlimineconf
mypkg-2
mypkg-3
```
> Note: As long as the templates are available for these packages in `myvoidrepo/void-packages/srcpkgs`, they will get built and added to your repository.

3. Update the `build_mypkgs` script with custom commands to pull the template and build the .xbps package for each entry in `packages.list`.

Example:
```bash
build_mypkgs() {
  case $1 in
    ...
    genlimineconf)
      # Pull latest srcpkg template
      curl -fSsL --create-dirs --output-dir="srcpkgs/genlimineconf" -o template "https://github.com/StrixROX/genlimineconf/releases/latest/download/srcpkg_template"

      # Build package from template
      ./xbps-src pkg genlimineconf
      ;;
    ...
  esac
}
```

4. Run the `update` script to pull the latest `void-packages` code, run a Void container, build the packages as per steps 2 & 3, register the packages in `myvoidrepo/repositry` and sign them. `update` calls all the other scripts (`build`, `sign`).

```bash
./myvoidrepo/update
```

> Note: `build` and `sign` scripts are not meant to be run independently. If you still want to, read through it and use at your own risk.

5. After `update` finishes, your signed packages and repository will be available in `myvoidrepo/repository` and you can serve it on your server.

# Serve Repository

Run the `serve` script to run an Apache server to serve your repository on port `8080`.

## Autostart Apache Server

1. Edit the provided `voidrepo-server.container.in` and replace `@HOST_PORT@` and `@HOST@` variables with your values and save to `voidrepo-server.container`.

> Replace @HOST@ with $HOST; replace @HOST_PORT@ with the port on the host you want to serve the repository on.

1. Use the resulting `voidrepo-server.container` to create a [podman quadlet](https://docs.podman.io/en/latest/markdown/podman-quadlet-basic-usage.7.html).

```bash
sudo podman quadlet install voidrepo-server.container           # installs voidrepo-server.service system unit (rootful podman container)
sudo systemctl daemon-reload                                    # reload systemd units
```

2. You can then enable the generated service to autostart the apache container on boot

```bash
sudo systemctl enable --now voidrepo-server
```

> Note: If you want to do this for a rootless container, you will have to create a [rootless podman quadlet]](https://docs.podman.io/en/latest/markdown/podman-quadlet-basic-usage.7.html#step-2-place-the-file), [enable lingering](https://wiki.archlinux.org/title/Systemd/User#Automatic_start-up_of_systemd_user_instances) for the user unit and enable the user service. Entirely too much hassle.
