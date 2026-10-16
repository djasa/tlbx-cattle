# `tlbx-cattle`

The _Container toolbx_ is for pet containers but pets get sometimes unwieldy. The `tlbx-cattle` comes to the rescue! Set up a `Containerfile` template, have it filled with variables defined in a config file, fire `tlbx-cattle` and have your containers created or rebuilt!

### Set up

Create config directories:
```
cp tlbx-cattle ~/.local/bin/

mkdir -p ~/.config/tlbx-cattle/{files,templates}
```

Add a custom file that you want to copy into containers:
```
cat > ~/.config/tlbx-cattle/files/my.repo << EOF
[my-repo]
name=My packages
baseurl=https://dnf.example.org
EOF
```

Create a jinja2 template for Containerfile:
```
cat > ~/.config/tlbx-cattle/templates/fedora << EOF
FROM registry.fedoraproject.org/fedora-toolbox:{{release}}
RUN hostname {{cont_name}}
COPY /my.repo /etc/yum.repos.d/
RUN dnf -y install {{pkgs}}
EOF

cat > ~/.config/tlbx-cattle/templates/private << EOF
FROM company.example.org/internal-image
RUN hostname {{cont_name}}
{{ my_lines }}
RUN dnf -y install {{pkgs}}
EOF
```

Only few variables for the template is defined by `tlbx-cattle`. Others are yours to fill in the container definitions in the config file using `var.`-prefixed keys in `cont` sections of the config file:

```
cat > ~/.config/tlbx-cattle/cattle.conf << EOF
[cattle]
parallel_builds = 4

[my_vars]
base_pkgs = black vim
my_pkgs = pkg1 pkg2
my_other_pkgs = pkg3 pkg4

[cont/fedora]
template = fedora
releases = 45 rawhide
var.pkgs = ${my_vars.base_pkgs} ${my_vars.my_pkgs}

[cont/special]
template = special
var.pkgs = ${my_vars.base_pkgs} ${my_vars.my_other_pkgs}
var.my_lines = COPY /internal-certs.pem /etc/pki/ca-trust/source/anchors/
    RUN update-ca-trust
EOF
```

#### Special variables

* `{{cont_name}}`: the final container name
* `{{release}}`: the release of the container (if `releases` is specified in the `cont/NAME` section)

### Run

```
tlbx-cattle
```

We end up with a trio of new containers, `fedora-45` and `fedora-rawhide` from `[cont/fedora]` config file section thanks to use of `releases` and `special` from `[cont/special]`. Enter one of them and try to do your stuff:
```
toolbox enter fedora-45
## your stuff
```

something is missing, you update your template or config and rebuild with `tlbx-cattle` again but then:
```
tlbx cattle
...
Containers rebuilt: fedora-rawhide special
Containers skipped: fedora-45
```

That is because the running containers are skipped unless called with `--force`. To stop the container, use `podman`:
```
podman stop fedora-45
```
We also skip building containers if jinja fails to fill your template.

Under the hood, we:
* pull new versions of base images
* create a new `localhost/CONT_NAME:build` image for each resulting container
* after success, we retag existing `localhost/CONT_NAME:current` as `:old`, then `:build` as `:current`
* then we remove current container and base new off of the `:current` image again
* last, we prune untagged images by `podman image prune --force`

This way, it's possible to recreate a container from the `:old` image in case it doesn't work and we don't occupy much more disk space than strictly needed.
