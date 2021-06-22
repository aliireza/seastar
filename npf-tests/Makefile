# Change the number of runs per experiment
N_RUNS?=5

# Default Configuration
MEMCACHED?=seastar
BENCHMRK?=ycsb
PROFILE?=pqos

# To avoid running multiple experiments at once
MAKEFLAGS := -j 1

# NPF Clusters (dut.node and pkt-gen.node should be defined in cluster/)
NPF_CLUSTER?=client=client-100G dut=dut-100G

NPF_OUTPUT_NAME?=kvs-results.pdf
NPF_OUTPUT= --output --output-columns x all --max-results --graph-filename ${NPF_OUTPUT_NAME}
NPF_INT_FLAGS := --show-full --rand-env 65536 --build-folder ./npf_build/ ${NPF_OUTPUT} $(NPF_FLAGS)
NPF_INT_FLAGS := $(NPF_INT_FLAGS) --force-retest

# Tools path 
SEASTAR_PATH=$(CURDIR)/.. # Seastar
MEMCACHED_SR_PATH=$(CURDIR)/../../bmc-cache/memcached-sr/ # Memcached SR
YCSB_PATH=$(CURDIR)/../../YCSB/ # YCSB
MUTILATE_PATH=$(CURDIR)/../../mutilate/ # Mutilate
EXP_PATH=$(CURDIR) # Current Directory

TOOLS_PATH := 
TOOLS_PATH += SEASTAR_PATH=${SEASTAR_PATH}
TOOLS_PATH += MEMCACHED_SR_PATH=${MEMCACHED_SR_PATH}
TOOLS_PATH += YCSB_PATH=${YCSB_PATH}
TOOLS_PATH += MUTILATE_PATH=${MUTILATE_PATH}
TOOLS_PATH += EXP_PATH=${EXP_PATH}



# Seastar Memcached with YCSB
seastar_ycsb: 
	npf-run local --testie memcached.npf --cluster ${NPF_CLUSTER} --tags ${MEMCACHED} ${BENCHMRK} ${PROFILE} coretune --config n_runs=${N_RUNS} ${NPF_INT_FLAGS} --variable ${TOOLS_PATH}
	rm -fr seastar_ycsb/
	mkdir -p seastar_ycsb
	mv kvs-results* seastar_ycsb/

# Seastar Memcached with Mutilate
seastar_mutilate: 
	npf-run local --testie memcached.npf --cluster ${NPF_CLUSTER} --tags ${MEMCACHED} mutilate ${PROFILE} coretune --config n_runs=${N_RUNS} ${NPF_INT_FLAGS} --variable ${TOOLS_PATH}
	rm -fr seastar_mutilate/
	mkdir -p seastar_mutilate
	mv kvs-results* seastar_mutilate/

# Memcached SR with YCSB
memcachedsr_ycsb: 
	npf-run local --testie memcached.npf --cluster ${NPF_CLUSTER} --tags sr ${BENCHMRK} ${PROFILE} coretune --config n_runs=${N_RUNS} ${NPF_INT_FLAGS} --variable ${TOOLS_PATH}
	rm -fr memcachedsr_ycsb/
	mkdir -p memcachedsr_ycsb
	mv kvs-results* memcachedsr_ycsb/