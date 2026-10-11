# `tlbx-cattle`

The _Container toolbx_ is for pet containers but pets get sometimes unwieldy. The `tlbx-cattle` comes to the rescue! Set up a `Containerfile` template, have it filled with variables defined in a config file, fire `tlbx-cattle` and have your containers created or rebuilt!

### Set up:

```
cp tlbx-cattle ~/.local/bin/

mkdir -p ~/.config/tlbx-cattle/{files,templates}

cat > ~/.config/tlbx-cattle/files/my.repo << EOF
[my-repo]
name=My packages
baseurl=https://dnf.example.org
EOF

cat > ~/.config/tlbx-cattle/cattle.conf << EOF
[cattle]
parallel_builds = 2

[my_vars]
base_pkgs = black vim
my_pkgs = pkg1 pkg2
my_other_pkgs = pkg3 pkg4

[cont/fedora]
template = fedora
releases = 45 rawhide
var.pkgs = ${my_vars.base_pkgs} ${my_vars.my_pkgs}

[cont/special]
template = fedora
releases = rawhide
var.pkgs = ${my_vars.base_pkgs} ${my_vars.my_other_pkgs}
EOF

cat > ~/.config/tlbx-cattle/templates/fedora << EOF
FROM registry.fedoraproject.org/fedora-toolbox:{{release}}
COPY /my.repo /etc/yum.repos.d/
RUN dnf -y install {{pkgs}}
EOF
```

### Run:

```
tlbx-cattle
```

These stopped containers will be rebuilt: `fedora-45`, `fedora-rawhide`, `special`, based off of images:
```
localhost/fedora-45:current
localhost/fedora-rawhide:current
localhost/special:current
```
Previous images are retagged as `:old` to enable reverting, new image is tagged `:build`. Old images are then pruned using `podman`.

### Special variables:

  * `{{release}}`: possible to specify as `--release` on CLI
