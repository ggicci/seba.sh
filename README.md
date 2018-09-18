# Sebastian

Build and ship a docker image without registry.

It's a script designed mostly for individual users and teams with small-scale IT infrastructure. We haven't got a private docker image registry. So we just build docker images on our development machines and scp them to remote servers for deployment.

## Quick Start

Here is a sample for demonstrating how to integrate `seba.sh` into a `Makefile`:

```Makefile
default: build

export IMAGE_NAME=example.com/helloworld
SEBA := "./seba.sh"

update-seba:
    curl --retry 3 --silent "https://raw.githubusercontent.com/ggicci/sebastian/master/seba.sh" --output seba.sh
    chmod u+x seba.sh

status:
    @$(SEBA) status

build:
    @$(SEBA) build

ship:
    @$(SEBA) ship --extra $(SEBA) Makefile --to \
        "ggicci@r01.example.com:/home/ggicci/somewhere" \
        "adm@r02.example-i.com(2222):"

install:
    @$(SEBA) install

deploy:
    docker-compose down
    @$(SEBA) install
    docker-compose up --no-start
    docker-compose start

logs:
    docker-compose logs --tail=50 -f
```

## Command List

```bash
Usage: seba [command]

command:
    status      show status
    build       build docker image
    save        save docker image and archive
    ship        ship docker images
    install     install docker images
    env         get value of seba variables

available seba variables:
    COMMIT, VERSION, SHIP_VERSION, IMAGE_NAME, IMAGE_TAR, IMAGE_TAR_GZ
```

### seba status

Show status of the current git repo. Sample output:

```
Commit: 5b525b9+CHANGES  Version: 2.7.0-31-g5b525b9  Ship: 2.7.0-31-g5b525b9
```

- Commit: current git commit number, "+CHANGES" will be applied if the repo is dirty;
- Version: human readable version (for build), you should use git tags for release versioning;
- Ship: version of last built docker image, ship command will scp image with this version to remote servers.

### seba build

Build docker image by using the default `Dockerfile` and pass two build args:

- `COMMIT`: see status "Commit"
- `VERSION`: see status "Version"

You **must** label them by `commit="${COMMIT}" version="${VERSION}"` in the `Dockerfile` in order to make seba working properly. Here is Dockerfile example:

```Dockerfile
FROM alpine:3.7

# seba build command will pass build args "COMMIT" and "VERSION"
ARG COMMIT
ARG VERSION

# "commit" and "version" labels are **necessary**
LABEL \
    name="example.com/helloworld" \
    url="https://github.com/exampledotcom/helloworld.git" \
    maintainer="..." \
    commit="${COMMIT}" \
    version="${VERSION}"
...
```

### seba save

Save the built image to a `.tar.gz`.

### seba ship

Save and then scp the `.tar.gz` file to remote servers.

### seba install

Load docker images from `.tar.gz` file to docker.

### seba env

Get value of a variable which is used in the seba script. e.g. `seba env VERSION` will echo the current version number.
