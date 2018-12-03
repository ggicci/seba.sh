# Sebastian

Build and ship a docker image without registry.

It's a script designed mostly for individual users and teams with small-scale IT infrastructure. We haven't got a private docker image registry. So we just build docker images on our development machines and scp them to remote servers for deployment.

## Quick Start

```shell
APP_NAME=com.example.helloworld IMAGE_NAME=example/helloworld ./seba.sh <sub_command>
```

Good practice for the naming conventions:

- APP\_NAME: `{reverse_domain}.{app_name}`, e.g. `org.example.helloworld`, `com.github.ggicci.sebastian`;
- IMAGE\_NAME: `{user_or_org_name}/{app_name}`, e.g. `example/helloworld`, `ggicci/sebastian`.


### Integrate with Makefile

```Makefile
default: build

export APP_NAME=org.example/helloworld
export IMAGE_NAME=example/helloworld

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
    dockerfile  show a template Dockerfile

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

Build docker image by using the default `Dockerfile` (run `seba dockerfile` to see a template of the Dockerfile) and pass three build args:

- `CREATED_AT`: current datetime in RFC3339 format
- `COMMIT`: see status "Commit"
- `VERSION`: see status "Version"

### seba dockerfile

Show a Dockerfile template. **Your Dockefile HAVE TO include the contents from this template.**

### seba save

Save the built image to a `.tar.gz`.

### seba ship

Save and then scp the `.tar.gz` file to remote servers.

### seba install

Load docker images from `.tar.gz` file to docker.

### seba env

Get value of a variable which is used in the seba script. e.g. `seba env VERSION` will echo the current version number.
