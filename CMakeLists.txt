# Copyright (c) Jazel Canseco, 2018
# Licensed under the Apache License, Version 2.0 (the "License")
cmake_minimum_required(VERSION 2.8.7)

project(bpfd)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 11)

set(CMAKE_C_FLAGS "-Wall")
set(CMAKE_CXX_FLAGS "-Wall")

IF(CMAKE_CROSSCOMPILING)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -static")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -static")
	set(lib_type STATIC)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O2 -s")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2 -s")
	set(LIBELF_LIBRARIES elf)
	set(ZLIB_LIBRARIES z)
	include_directories(/home/share/elf-lib-arm/output/include)
	link_directories(/home/share/elf-lib-arm/output/lib)
ELSE()
	find_package(ZLIB REQUIRED)
	find_package(LibElf REQUIRED)
	set(lib_type SHARED)
ENDIF()

include_directories(src)
include_directories(bcc-deps)
include_directories(bcc-deps/compat)

add_library(bpfd-bcc ${lib_type} bcc-deps/libbpf.c bcc-deps/perf_reader.c
	bcc-deps/bcc_elf.c bcc-deps/bcc_perf_map.c bcc-deps/bcc_proc.c
	bcc-deps/bcc_syms.cc bcc-deps/ns_guard.cc bcc-deps/common.cc
	bcc-deps/vendor/tinyformat.hpp)
target_link_libraries(bpfd-bcc ${LIBELF_LIBRARIES} ${ZLIB_LIBRARIES})

add_executable(bpfd src/bpfd.c src/base64.c src/remote_perf_reader.c src/utils.c
			src/cmd_parsers.c)
target_link_libraries(bpfd bpfd-bcc)
