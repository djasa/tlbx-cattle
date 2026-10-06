# `toolbox-custom`

A helper to customize and regenerate images for toolbox.

### Set up:

```
cp toolbox-custom ~/.local/bin/

mkdir -p ~/.config/toolbox-custom/files

cat > ~/.config/toolbox-custom/files/my.repo << EOF
[my-repo]
name=My packages
baseurl=https://dnf.example.org
EOF

cat > ~/.config/toolbox-custom/toolbox-custom.conf << EOF
pkgs = git vim

[vars]
my_pkgs = pkg1 pkg2
EOF

cat > ~/.config/toolbox-custom/fedora << EOF
FROM registry.fedoraproject.org/fedora-toolbox:{{release}}
ADD /my.repo /etc/yum.repos.d/
RUN dnf -y install {{my_pkgs}}
```

### Run:

check the `Containerfile` to be built:
```
toolbox-custom fedora --release 45 --print
```

(re)build the image
```
toolbox-custom fedora --release 45
```

The old image of the same name gets deleted and new one gets created (in this order for the time being)

### Special variables:

  * `{{release}}`: possible to specify as `--release` on CLI

It's also possible to set or override any variable from CLI:

```
toolbox-custom fedora --release 45 --var "my_pkgs:pkg1 pkg2 pkg3"
```
