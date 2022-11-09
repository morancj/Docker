# Docker OpenSSH client for ancient targets

**This is a crash course, not a complete overview of Docker usage!**

## Building the image

Build with e.g:

```shell
$ docker build --rm --tag "ubuntu/ssh:14.04" .
```

## Get basic info about the image

```shell
$ docker image ls ubuntu/ssh:14.04
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
ubuntu/ssh   14.04     c3f767d1138c   4 seconds ago   201MB
```

## Running the image

Run with e.g:

```shell
$ docker run \
  --cap-drop=all \
  --memory=1GB \
  --rm \
  -t \
  -i \
  -e SSH_AUTH_SOCK="${SSH_AUTH_SOCK}" \
  --mount type=bind,src=$SSH_AUTH_SOCK,dst="${SSH_AUTH_SOCK}",readonly=false \
  --mount type=bind,src="$HOME/.ssh",dst="$HOME/.ssh",readonly=true \
  --mount type=bind,src=/etc/group,dst=/etc/group,readonly=true \
  --mount type=bind,src=/etc/passwd,dst=/etc/passwd,readonly=true \
  -u "$(id -u)":"$(id -g)" \
  --userns=host \
  --hostname $(hostname)-ubuntu-1404-ssh \
  "ubuntu/ssh:14.04" $user@$hostname
```

replacing `$user` and `$hostname` with the appropriate values.

### Command protip

Make this massive Docker command a local function with something like this:

```shell
function oldssh(){
  docker run \
  --cap-drop=all \
  --memory=1GB \
  --rm \
  -t \
  -i \
  -e SSH_AUTH_SOCK="${SSH_AUTH_SOCK}" \
  --mount type=bind,src=$SSH_AUTH_SOCK,dst="${SSH_AUTH_SOCK}",readonly=false \
  --mount type=bind,src="$HOME/.ssh",dst="$HOME/.ssh",readonly=true \
  --mount type=bind,src=/etc/group,dst=/etc/group,readonly=true \
  --mount type=bind,src=/etc/passwd,dst=/etc/passwd,readonly=true \
  -u "$(id -u)":"$(id -g)" \
  --userns=host \
  --hostname $(hostname)-ubuntu-1404-ssh \
  "ubuntu/ssh:14.04" "${1}"
}
```

Use it with `oldssh $user@$host`.

There are smarter ways to do this, including sharing common options, etc, but
this will get you started.

### SSH protip

If you have a GitHub account and your SSH key loaded, you can use
`git@github.com` for `$user@$hostname`. If you see something like this, you have
correctly authenticated:

```shell
...snip...
Failed to add the ECDSA host key for IP address '140.82.121.3' to the list of known hosts (/home/username/.ssh/known_hosts).
PTY allocation request failed on channel 0
Hi $your_github_username! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.
```

The failures are
1. Because we mounted "${HOME}/.ssh" read-only, and
2. Because GitHub doesn't allocate a TTY.

These are to be expected.

## Terminating

On a clean exit, this should clean up after itself. If you accidentally or
deliberately crash it, you may have stopped containers on your system.

To check, use `docker ps -a`. If you see it, you can remove like this:

```shell
$ docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                        PORTS     NAMES
89dcc9c3a82b   aaa343628a70   "/bin/sh -c 'export …"   30 minutes ago   Exited (100) 29 minutes ago             silly_johnson


$ docker rm 89dcc9c3a82b
```

You should see the container removed in the output of any subsequent
`docker ps -a`.

## Removing the image

Remove with `docker rmi ubuntu/ssh:14.04`.

Check with `docker image ls`.