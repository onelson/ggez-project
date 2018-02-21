The docker-compose config included in the repo, along with some
supporting scripts should help bootstrap a development environment.


# How to use

From a fresh clone, run:

```
$ ./setup.sh
...
```

The containers are created using _the current user's_ **uid** and
**gid** so be sure to run this as you (not root). This is critical
to making files generated inside the containers have the correct
ownership.

# Develop

The provided `docker-compose.yml` is geared towards developing rust programs 
that may require native dependencies, allowing those deps to be explicitly 
stated (by the Dockerfile).

The compose config manages a data volume for the `CARGO_HOME` so you can 
install deps and tools as needed and they will be preserved from run to run.

If you need to wipe your container clean, use `docker-compose down --volumes`
then re-run `./setup.sh` to rebuild your image.

Your development will happen inside the container, and as such you may like
to use the `./run.sh` script as a shorthand way to execute commands. For 
example:

```
$ ./run.sh cargo check
...
```

For running blocking commands that do funny stuff to the signal handling
(like `watchexec` when run inside the docker container) there is another helper 
script called `./watch.sh` which does some extra work to ensure you can kill
the command with `SIGTERM` as you'd hope. For example:

```
$ ./run.sh cargo install watchexec
...

$ ./watch.sh watchexec cargo test
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/my_binary-068139cf154b28dd

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

^C
$
```

A regrettable side-effect of how `./watch.sh` works is it'll strip any color 
information out of the command output. Sorry! Hopefully there's a better 
solution out there for this and we'll find it eventually.

The build in the `dev` image is dynamicly linked and as such, native deps can 
be installed via `apt-get`.

# Notable Things That Are Missing

## Sound

While we've been able to expose the graphics hardware on the host to the 
docker container, we have not yet gotten sound working correctly. For now, 
your game will run, but with a default sound device of "null" (note the 
cheeky hack at the tail of the dev Dockerfile).

## Windows Support

Part of the deal with this docker compose config is that the docker image is 
meant to stamp the current user's `uid` and `gid` on to the user that executes
things inside the container. This only makes sense when running on a host OS 
that works with posix groups (like Linux or Darwin). It might be made to work
for Windows, but it won't without some additional work.

## Mac Support???

This hasn't been tested on Darwin, and there's a chance it will just work, but
there's a chance is just won't.

The graphics hardware is shared with the container by way of exposing the X11
socket file from the host, and it would seem to me this is probably not how 
Macs work...

## Releases

I have plan to investigate setting up a Dockerfile for doing cross-builds 
(for Windows) and static binaries for releases, but this is still TODO.

Likely the native deps (SDL2 and libalsa, aka asound) will have to be 
compiled from source so they will work with the musl cross-build process.
