# ODSL computing at MPCDF

## Resources at MPCDF

The Max-Planck-Institute for Physics (MPP) operates a compute cluster (MPP-Cluster) hosted at MPCDF that offers both a SLURM batch system for batch jobs and nodes for interactive use. In addition, MPCDF hosts the supercomputers Cobra and Raven that MPCDF members can apply for access to.


## Client setup

You cannot access the various MPCDF systems directly from outside of the MPCDF and MPP networks, instead, your have to connect via `gate1.mpcdf.mpg.de` and `gate1.mpcdf.mpg.de`.

See the [ssh-client-config](ssh-client-config) file for a suggested SSH client configuration on your **local** system (laptop, desktop PC, etc.) that uses SSH ProxyJump simplify this. The configuration also uses SSH ControlMaster keep a connection alive in the background, so your don't have to enter your password every time when connection to an MPCDF host repeatedly.

Now your should be able to do things like

```shell
ssh mpp-cluster
ssh odslserv02
```


## The MPP-Cluster at MPCDF

The cluster offers

* The interactive head nodes `mppui1.t2.rzg.mpg.de`, `mppui2.t2.rzg.mpg.de` and `mppui4.t2.rzg.mpg.de`.

* A set of compute nodes for SLURM batch jobs.

* Interactive high-performance CPU and CPU+GPU nodes:

* A cluster file system "/ptmp" (GPFS) that is available on all nodes.

* A dCache storage system together with the relevant user-space tools (relevant for HEP projects).
  
* Support for [Apptainer](https://apptainer.org/) software containers.

For historical reasons, older nodes in the MPCDF MPP cluster are part of the domain `t2.rzg.mpg.de`, while newer nodes are part of `t2.mpcdf.mpg.de`.


### Interactive compute nodes (compute servers)

The MPP-Cluster has interactive compute nodes, some of which are dedicated to specific projects:

* odslserv01.t2.mpcdf.mpg.de (dedicated to ODSL): 2x64 CPU cores, 4x NVIDIA A100 GPU (with NVLink), 1TB RAM

* odslserv02.t2.mpcdf.mpg.de (dedicated to ODSL): 2x48 CPU cores, 4x NVIDIA H100 GPU (with NVLink), 1TB RAM

More interactive compute nodes will be added in the future.

The interactive compute nodes are intended for CPU and GPU code development, benchmarking, interactive data analysis and short compute runs. If you do do run stuff in the background for longer than a few minutes, please use `nice -n 15` or similar to ensure interactive use has priority.

In addition to the network files-systems listed above, the following node-local file systems are available here:

* "/tmp": Typical temp directory, but rather large and on fast solid state disks. This directory is automatically cleared regularly.

* "/scratch": A large local file system on fast solid state disks. Unlike "/tmp", this directory is not cleared regularly, but may be cleared from time to time with (or possibly even without) short-term advance warning.


### MPP-Cluster home directory setup

#### Setting up `cenv` container environments

Instead of installing your software manually, we recommend that you use Apptainer software containers. A default ODSL container image with standard machine learning software is available. We recommend that you use [cenv](https://github.com/oschulz/container-env) to set up and run your container environments. `cenv` is mostly a wrapper for `apptainer` that automatically takes care of redirecting various Python, Jupyter and Julia environment variables, as well as enabling Apptainer GPU support automatically if GPUs are available on the host.

Run

```shell
ln -s "/ptmp/mpp/software/container-env/bin/cenv" "$HOME/.local/bin"
```

to add `cenv` to `$HOME/.local/bin` (should be on your `$PATH`).

By default, `cenv` will store it's container environments in `$HOME/.cenv`. This is not ideal on the MPCDF-cluster, since the home directories are currently on AFS and distributed performance is low. So you may want to redirect your `.cenv` directory to
your personal directory on `/ptmp`:

```shell
test -e "$HOME/.cenv" && rmdir "$HOME/.cenv" # should be empty or not exist yet
ln -s "/ptmp/mpp/`whoami`/cenv" "$HOME/.cenv"
```

Now create an `cenv` environment named `odsl` that uses the standard ODSL Apptainer image:


```shell
cenv --create odsl /ptmp/mpp/software/apptainer/images/odsl/mppmu_odsl-ml_latest.sif
```

This will create a new directory `.cenv/odsl` that contains a symbolic link to the Apptainer image (you can also create this manually).

Now you can just run

```shell
cenv odsl
```

any time that you want to enter an ODSL container instance. Inside such a container instance, `/homedir/` points to `$HOME` and `/user` points to `$HOME/.cenv/CENV_NAME`, allowing you to use these paths in a portable fashion. Run

```
export | grep '/user'
```

inside the cenv container instance to list the environment variables that `cenv` redirected to paths under `/user` by default.

Run

```shell
cenv --help
```

for more information. If you used the SSH client configuration linked above, you should now also be able to SSH directly into a `cenv` container instance:

```
ssh odsl~odslserv02
```

See the  [cenv VS-Code documentation](https://github.com/oschulz/container-env/blob/main/README-VSCode.md) on how to use this to use remote development in `cenv` container environments.


## The MPCDF supercomputers

The MPCDF supercomputers Cobra and Raven are available to ODSL members on request, for moderate use, without a formal project-specific application. Intense use of these HPC resources needs to be discussed first, though.

See the [MPCDF computing documentation](https://docs.mpcdf.mpg.de/doc/computing/index.html) regarding technical information.
