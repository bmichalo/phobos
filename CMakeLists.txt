CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
PROJECT(phobos C CXX)

SET(CMAKE_CXX_FLAGS "-fno-stack-protector -Wall -Wextra -Wno-unused-parameter -g -O3 -std=gnu++11 -march=native -msse4.2")
SET(CMAKE_C_FLAGS "-fno-stack-protector -Wall -Wextra -Wno-unused-parameter -g -O3 -std=gnu11 -march=native -msse4.2")
SET(CMAKE_EXE_LINKER_FLAGS "-rdynamic") # to access functions from luajit

SET(FILES
	src/main
	src/memory
	src/task
	src/device
	src/i40e
	src/util
	src/lifecycle
	src/barrier
	src/task-results
	src/pipe
	src/lock
	src/namespaces
	src/ring
	src/kni
	src/filter
	src/pcap
	src/timestamping
	src/timestamping_i40e
	src/timestamping_ixgbe
	src/timestamping_igb
)

SET(DPDK_LIBS
	ethdev       rte_distributor  rte_kni       rte_mbuf      rte_pmd_bond   rte_pmd_ring         rte_power  rte_timer
	rte_acl      rte_eal          rte_kvargs    rte_mempool   rte_pmd_e1000  rte_pmd_virtio       rte_ring
	rte_cfgfile  rte_hash         rte_lpm       rte_meter     rte_pmd_i40e   rte_pmd_vmxnet3_uio  rte_sched
	rte_cmdline  rte_ip_frag      rte_pipeline  rte_pmd_ixgbe rte_port       rte_table
)

SET(LUAJIT_LIBS
	luajit-5.1
)

SET(ALL_LIBS
	${LUAJIT_LIBS}
	-Wl,--start-group ${DPDK_LIBS} -Wl,--end-group
	pthread dl rt
)

INCLUDE_DIRECTORIES(
	${CMAKE_CURRENT_SOURCE_DIR}/deps/dpdk/x86_64-native-linuxapp-gcc/include
	${CMAKE_CURRENT_SOURCE_DIR}/deps/dpdk/drivers/net/ixgbe/base
	${CMAKE_CURRENT_SOURCE_DIR}/deps/dpdk/drivers/net/ixgbe
	${CMAKE_CURRENT_SOURCE_DIR}/deps/dpdk/drivers/net/i40e/base
	${CMAKE_CURRENT_SOURCE_DIR}/deps/dpdk/drivers/net/i40e
	${CMAKE_CURRENT_SOURCE_DIR}/deps/dpdk/drivers/net/e1000
	${CMAKE_CURRENT_SOURCE_DIR}/deps/dpdk/drivers/net/e1000/base
	${CMAKE_CURRENT_SOURCE_DIR}/deps/luajit/src
	${CMAKE_CURRENT_SOURCE_DIR}/lib
	${CMAKE_CURRENT_SOURCE_DIR}/src
)

LINK_DIRECTORIES(
	${CMAKE_CURRENT_SOURCE_DIR}/deps/dpdk/x86_64-native-linuxapp-gcc/lib
	${CMAKE_CURRENT_SOURCE_DIR}/deps/luajit/usr/local/lib
)

IF(PHOBOS_BUILD_LIBRARY)
	ADD_DEFINITIONS(-DPHOBOS_BUILD_LIB)
	ADD_LIBRARY(phobos STATIC ${FILES})
ELSE()
	ADD_EXECUTABLE(phobos ${FILES})
ENDIF()

TARGET_LINK_LIBRARIES(phobos ${ALL_LIBS})

