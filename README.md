<pre>
     /\   | |    |  __ \
    /  \  | |    | |__) |
   / /\ \ | |    |  ___/
  / ____ \| |____| |
 /_/___ \_\______|_|
 |  __ \                        | |          (_) |   | |    |  _ \      (_) |   | |    
 | |__) |___ _ __  _ __ ___   __| |_   _  ___ _| |__ | | ___| |_) |_   _ _| | __| |___ 
 |  _  // _ \ '_ \| '__/ _ \ / _` | | | |/ __| | '_ \| |/ _ \  _ <| | | | | |/ _` / __|
 | | \ \  __/ |_) | | | (_) | (_| | |_| | (__| | |_) | |  __/ |_) | |_| | | | (_| \__ \
 |_|  \_\___| .__/|_|  \___/ \__,_|\__,_|\___|_|_.__/|_|\___|____/ \__,_|_|_|\__,_|___/
            | |
            |_|
</pre>


# ReproducibleBuilds
A collection of `Dockerfile`s as well as associated guides and documentation
that together facilitate reproducible builds of ALP projects. These can in
particular be used within a Continuous Integration (CI) setup, but also a simple
mean to have a safe, reproducible environment to debug and play with ALP.

The main Docker image (`Dockerfile.lpf-ubuntu-20.04-gcc`) contains the following
software:

- basic development tools and dependencies for ALP/GraphBLAS and LPF: `g++`,
  `gcc`, `libnuma`, `mpich`, `make`, `ninja`, `lcov` for coverage
- CMake version 3.13, directly callable from the command line as `cmake`
- CMake 3.26.1 (latest version at the time of writing), located at
  `${CMAKE_RECENT}`
- LPF, located in the path `${LPF_PATH}`
- ALP/GraphBLAS standard testing datasets, to be previously downloaded through
  the [downloadDatasets.sh script](downloadDatasets.sh)

Everything is in `/home/alp_ci/alp_deployment`:
- `cmake` deployment
- `cmake_3.26.4` deployment  
- LPF deployment
- `datasets` ALP/GraphBLAS standard testing datasets

A second image (`Dockerfile.lpf-ubuntu-22.04-gcc-clang`) follows the same
structure, but installs both GCC and Clang, both in multiple versions in order
to better test ALP code compliance. Consequently, it compiles LPF for each
version and compiler.


# Content of this repo
- this `README.md` with basic info
- the `Dockerfile.lpf-ubuntu-20.04-gcc` to build the main image
- the `Dockerfile.lpf-ubuntu-22.04-gcc-clang` to build the image with more
  compilers
- the `downloadDatasets.sh` script to download ALP standard testing script
  (subject to license acceptance)


# Docker build arguments
The main image is built with the following arguments (with defaults)

- `TIMEZONE=Etc/UTC` timezone for the installation of dependencies

The image with more compilers may contain more arguments: you may refer to the
internal comments to understand their goal.

# Build the main image
The first step is to download the datasets, for which you should accept the
related licenses. In the folder of this repository, you may simply run:

```bash
echo "yes" | ./downloadDatasets.sh
```

to accept the license and proceed to the downloads.
Now the datasets are available in the `datasets` directory, and the container
can be built

## Build for your own machine
To simply build the image for your own machine with tag `lpf-ubuntu-20.04-gcc`:

```bash
docker build -t lpf-ubuntu-20.04-gcc -f Dockerfile.lpf-ubuntu-20.04-gcc .
```

## Build and push to a Docker registry
To build for your own Docker registry (e.g., the CI Docker registry), you need
to know the Docker registry URL, hereon indicated as `<Docker registry URL>`:

1. first login into the registry, if you have not done so yet
    ```bash
    docker login <Docker registry URL>
    ```
2. then you should build the image with the proper name, matching that of the
   registry (e.g., for GitLab
   `<Docker registry URL>/<group name>/<project name>/` --
   see GitLab documentatio
   [here](https://docs.gitlab.com/ee/user/packages/container_registry/build_and_push_images.html));
   e.g.:
    ```bash
    docker build -t <Docker registry URL>/<group name>/<project name>/lpf-ubuntu-20.04-gcc -f Dockerfile.lpf-ubuntu-20.04-gcc .
    ```
3. finally, you may push the image to the registry
    ```bash
    docker push <Docker registry URL>/<group name>/<project name>/<possible sub-name>/lpf-ubuntu-20.04-gcc
    ```

within the containers, all commands run by default in an empty directory
`/alp_ci`, which is different from the one where dependencies and tools are
deployed; these are usually in `/alp_deployment` and can be reached via the
dedicated environment variables (see the `ENV` directives in the `Dockerfile`s).

Note: processes inside the container **run as root**, in order to be able to
install custom dependencies easily. No limited user privilege is enforced.

# Other images
The file `Dockerfile.lpf-ubuntu-22.04-gcc-clang` builds a Docker image from
Ubuntu 22.04, similarly to what described above. However, Ubuntu 22.04 offers
more compiler versions for both GCC and Clang, which are stored in this image.
The offered versions are stored in the environment variables `GCC_VERSIONS` and
`CLANG_VERSIONS`, respectively. All related LPF deployments are built from these
compilers, under the following naming scheme:

- `${LPF_BASE_PATH}/build_mpich_gcc_9/install` deployment for LPF built with MPICH and GCC 9
- `${LPF_BASE_PATH}/build_mpich_gcc_10/install` deployment for LPF built with MPICH and GCC 10
- ...
- `${LPF_BASE_PATH}/build_mpich_clang_11/install` deployment for LPF built with MPICH and Clang 11
- `${LPF_BASE_PATH}/build_mpich_clang_12/install` deployment for LPF built with MPICH and Clang 12
- ...

Users may iterate over `GCC_VERSIONS` and `CLANG_VERSIONS` to access these
deployments, e.g.: `for ver in $GCC_VERSIONS; do ... done;`.

**Note**: due to several bugs, this image uses a "patched" LPF versions stored
in the branch `LPF_BRANCH_NAME`, which is an optional build argument for the
`docker build` command.

This image follows the same building process as the main one and also needs the
datasets.
