# Running a simple container with runc

For this example, we need a linux filesystem.

Make and enter the working directory.
```sh
$ mkdir example
$ cd example
```

Create the filesystem
```sh
$ docker export $(docker create alpine) > alpine.tar
$ mkdir rootfs
$ tar -C rootfs -xf alpine.tar
```

For this example, we will run rootless. This allows for containers to be created without privileges.
```sh
$ runc spec --rootless
```

Run the container
```sh
$ runc run foo
```

We are now in the container.

A few caveats:

* The filesystem is **read-only**
* The network namespace is the same as the invoking process. If you're not already running in a container, this is likely your host network.
