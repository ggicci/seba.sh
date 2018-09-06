# Sebastian

A tiny tool for **docker applications** including build, deploy, etc.

## Quick Look

A sample project:

```
helloworld
├── Dockerfile
├── Makefile
├── docker-compose.yml
├── seba.sh
└── ... (others)
```

Example Dockerfile:

```Dockerfile
FROM alpine:3.7

# seba build command will pass build args "COMMIT" and "VERSION"
ARG COMMIT
ARG VERSION

# "commit" and "version" labels are necessary
LABEL \
    name="example.com/helloworld" \
    url="https://github.com/exampledotcom/helloworld.git" \
    maintainer="..." \
    commit="${COMMIT}" \
    version="${VERSION}"
...
```

Example Makefile:

```Makefile
default: build

export IMAGE_NAME=example.com/helloworld
SEBA := "./seba.sh"

status:
	@$(SEBA) status

build:
	@$(SEBA) build

ship:
	@$(SEBA) ship \
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

You **must** label them by `commit="${COMMIT}" version="${VERSION}"` in the `Dockerfile` in order to make seba working properly.

### seba save

Save the built image to a `.tar.gz`.

### seba ship

Save and then scp the `.tar.gz` file to remote servers.

### seba install

Load docker images from `.tar.gz` file to docker.
