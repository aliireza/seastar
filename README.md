Seastar (with MLX5 PMD)
=======

This repo contains information+fixes to run Seastar (specifically memcached) with DPDK and Mellanox ConnectX-5 NIC driver (i.e., MLX5 PMD).

# Building Seastar
--------------------

For more details and alternative work-flows, read the original [README.md](./README.md.old) and [HACKING.md](./HACKING.md).

## Dependencies

### gcc/g++

Seastar needs gcc/g++ 10. You can set it up as follows:

```bash
sudo apt install software-properties-common
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update
sudo apt-get install gcc-10 g++-10
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-10 60 --slave /usr/bin/g++ g++ /usr/bin/g++-10
sudo update-alternatives --config gcc
```

**You have to select gcc 10 as your default.**

### MLX5 Dependencies

MLX5 driver requires `libibverbs` and `libmlx5` libraries. Additionally, you have to install Mellanox OFED. To ensure that you do not miss the MLX5 dependencies, please build DPDK independetly and run `test_pmd` application. 


### Seastar Dependencies

Seastar requires some libraries such as `libboost`, `fmt`, and `hwloc`. You can install them manually, but you might encounter some issues, due to wrong version. You can use Seastar's `configure.py` script to download and locally install these dependencies.

## Building

To build Seastar, run the following commands. 

```bash
git clone https://github.com/aliireza/seastar.git
git submodule update --init
sudo ./install-dependencies.sh
./configure.py --mode=release --enable-dpdk --cook Boost --cook fmt --cook hwloc --cook dpdk
ninja -C build/release
```

Note that you might still get some errors, due to missing packages/libraries. Most of them can be solved via `apt-get`, but you can also add extra `--cook` flags to resolve extra dependencies, check `./cooking.sh` for more info.

**Since using `--cook` calls the `./cooking.sh`, you can manually check for errors if a `--cook` command fails. For example, try `./cooking.sh -e Boost` or `./cooking.sh -e hwloc`.**

## Running Memcached

To run Memcached with DPDK + native userspace TCP stack, run the following command:

```bash
sudo build/release/apps/memcached/memcached --network-stack native --dpdk-pmd --dhcp 0 --host-ipv4-addr 192.168.101.13 --netmask-ipv4-addr 255.255.255.0 --collectd 0 --cpuset 0,2 --dpdk-port-index 0 --thread-affinity 1
```

- `--cpu-set` specifies the exact cores to be used by Seastar, but you can also use `--smp` to set the number of cores. Note that we have also used `--thread-affinity 1`. 
- You can change the `--dpdk-port-index` to select the appropriate NIC/port.
- You can add `--stats` to print server statistics. Check memcached help for more info: `sudo build/release/apps/memcached/memcached -h`