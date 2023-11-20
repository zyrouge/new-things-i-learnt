# Lauching Linux applications inside a Podman container

## History

I could not run a Flutter application since it did not build on Fedora 39.
So, I ended up using Ubuntu 22.04 to build and run the application.

## How did I do it?

1. I pulled the Ubuntu 22.04 image required.
2. When creating/launching the container using the image, pass necessary parameters and variables so it can access the native resources.

```bash
podman run --rm -it \
    --userns keep-id \
    --no-hosts \
    --network host \
    --security-opt label=type:container_runtime_t \
    --name "<container_name>" \
    -v "/tmp/.X11-unix:/tmp/.X11-unix:ro" \
    -v "$XAUTHORITY:$XAUTHROITY:ro" \
    -v "$XDG_RUNTIME_DIR:$XDG_RUNTIME_DIR" \
    -e DISPLAY \
    -e WAYLAND_DISPLAY \
    -e XAUTHORITY \
    -e XDG_RUNTIME_DIR \
    ...
    -v "<host_fs_entity>:<container_fs_entity>:z" \
    -w "<working_directory>"
    "<image_name>"
```

## How does this work?

What we are essentially doing is, we pass the display socket with it's authentication protocol.

First, we'll focus of supporting X11 only, this is done using the below.

- Mounting `/tmp/.X11-unix` gives access to the X11 display socket. This is same for most linux distributions.
- Mounting `$XAUTHORITY` is what grants access to the usage of the X11 socket. If you don't pass it, you'd get an authorization error.
- Passing `$DISPLAY` says which display to use. If you only have a single screen, it would be `:0`.

You can run X11 applications with just this. But, if you need Wayland, you need to do more work.
Wayland is complex, but the basic functionality can be achieved by the below.

- Mounting `$XDG_RUNTIME_DIR` gives access to resources such as Wayland socket and other crucial resources such as Pipewire. Remember this directory contains some sensitive things. Considering mounting only required subdirectories.
- Passing `$WAYLAND_DISPLAY` says which display to use too. This is a Wayland way to doing it.

I did get Wayland applications to launch and work smoothly. But, things like theming were not being inherited.
You might need to expose more of your filesystem if you need it.
You should probably consider Distrobox or Toolbx for easier way to do those.

What are other flags for?

- `--userns keep-id` is an easier way to map the same uid and gid of host to the container. This is required for display authentication to work. It probably also required to access resources in `$XDG_RUNTIME_DIR`.
- `--no-hosts` is used to ask Podman not to create `/etc/hosts`. I'm not sure how crucial this is, but it doesn't hurt anything.
- `--network host` allows Podman to expose host's network resources to the container. This is how access to sockets are provided.
- `--security-opt label=type:container_runtime_t` is a security option that is related to SELinux. I'm not sure what this does either. But, this seems to be required.

Hope this helps! :)
