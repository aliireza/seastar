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

## Running Benchmark

This section summarizes some of popular benchmarks for memcached. For offical Seastar Memcached benchmarking guidelines refer to [here][seastar-memcached-benchmark].

### Memslap/Memaslap (libmemcached)

You can use [memslap][memslap-page] and/or [memaslap][memaslap-page], which are popular tools provided by [libmemcached][libmemcached-page], to benchmark a memcached server. To setup memaslap, run the following commands:

```bash
sudo apt-get install python3-sphinx bison flex libevent-dev
wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz
tar -xvf libmemcached-1.0.18.tar.gz
cd libmemcached-1.0.18/
./configure --enable-memaslap
make 
sudo make install
```

- If you could not find `memaslap`, check [here][libmemcached-bugs].

- You might need to fix some compilation errors, depending on your compiler version.

To run memaslap, you can run the following command:

```bash
memaslap -s 192.168.101.13:11211 -t 60s -T 1 -c 60 -X 64
```

- If you want to use UDP, you should also pass `-U` flag. 


### YCSB

You can use [YCSB][ycsb-repo] to benchmark memcached. To setup YCSB, you can run the following commands:

```bash
sudo apt install maven 
git clone http://github.com/brianfrankcooper/YCSB.git
cd YCSB
mvn -pl site.ycsb:memcached-binding -am clean package
```

A sample command to run YCSB could be: `./bin/ycsb load memcached -s -P workloads/workloada -p "memcached.hosts=192.168.101.13"` 

**For more information, please refer to this [page][ycsb-memcached].**


### Mutilate

[Mutilate][mutilate-page] is a open-loop load generator for memcached. To setup mutilate, run the following commands:

```bash
sudo apt-get install scons libevent-dev gengetopt libzmq3-dev
git clone https://github.com/leverich/mutilate.git
cd mutilate/
scons
```

To run mutilate, you can run the following command:

```bash
./mutilate -s 192.168.101.13 -T 16 -c 32 -K 8 -V 1024 -w 5 -u 0.1 -t 60
```

### Cloud Suite (Data Caching)

You can also use [cloud suite][cloud-suite-page] to benchmark memcached by using Twitter dataset.

You can setup cloud-suite benchmarking tool via docker containers or by running the following commands:

```bash
wget https://github.com/parsa-epfl/cloudsuite/raw/master/benchmarks/data-caching/client/memcached.tar.gz
tar -xvf memcached.tar.gz
cd memcached/
make
```

For more information, please refer to [their repository][cloud-suite-data-caching].

[seastar-memcached-benchmark]: https://github.com/scylladb/seastar/wiki/Memcached-Benchmark
[memslap-page]: http://docs.libmemcached.org/bin/memslap.html
[memaslap-page]: http://docs.libmemcached.org/bin/memaslap.html
[libmemcached-page]:https://libmemcached.org/libMemcached.html
[libmemcached-bugs]: https://bugs.launchpad.net/libmemcached/+bug/1562677
[ycsb-repo]: https://github.com/brianfrankcooper/YCSB
[ycsb-memcached]: https://github.com/brianfrankcooper/YCSB/tree/master/memcached
[mutilate-page]: https://github.com/leverich/mutilate
[cloud-suite-page]: https://www.cloudsuite.ch/
[cloud-suite-data-caching]: https://github.com/parsa-epfl/cloudsuite/blob/master/docs/benchmarks/data-caching.md
