cmake_minimum_required(VERSION 3.16 FATAL_ERROR)

set(LIBVERSION "17:0:6")
set(STDC_HEADERS 1)

project(libnftnl VERSION 1.2.9 LANGUAGES C)

if(NOT CMAKE_SYSTEM_NAME STREQUAL "Linux")
    message(FATAL_ERROR "Oops! libnftnl is only supported on Linux systems.")
endif()

option(BUILD_TESTS "Build tests" OFF)
option(BUILD_EXAMPLES "Build examples" OFF)
option(BUILD_DOC "Build documentation" OFF)

# Find dependencies
find_package(PkgConfig REQUIRED)
pkg_check_modules(LIBMNL REQUIRED libmnl>=1.0.4)

# Check for required libraries
include(CheckIncludeFile)
check_include_file(dlfcn.h HAVE_DLFCN_H)
check_include_file(inttypes.h HAVE_INTTYPES_H)
check_include_file(stdint.h HAVE_STDINT_H)
check_include_file(stdio.h HAVE_STDIO_H)
check_include_file(stdlib.h HAVE_STDLIB_H)
check_include_file(strings.h HAVE_STRINGS_H)
check_include_file(string.h HAVE_STRING_H)
check_include_file(sys/stat.h HAVE_SYS_STAT_H)
check_include_file(sys/types.h HAVE_SYS_TYPES_H)
check_include_file(unistd.h HAVE_UNISTD_H)

# Check for visibility support
include(CheckCCompilerFlag)
check_c_compiler_flag(-fvisibility=hidden HAVE_VISIBILITY_HIDDEN)

# Generate config.h
configure_file(config.h.cmake.in ${CMAKE_BINARY_DIR}/config.h)

# Create library
add_library(nftnl)

target_sources(nftnl 
    PRIVATE
        src/utils.c
        src/batch.c
        src/flowtable.c
        src/common.c
        src/gen.c
        src/table.c
        src/trace.c
        src/chain.c
        src/object.c
        src/rule.c
        src/set.c
        src/set_elem.c
        src/str_array.c
        src/ruleset.c
        src/udata.c
        src/expr.c
        src/expr_ops.c
        src/expr/bitwise.c
        src/expr/byteorder.c
        src/expr/cmp.c
        src/expr/range.c
        src/expr/connlimit.c
        src/expr/counter.c
        src/expr/ct.c
        src/expr/data_reg.c
        src/expr/dup.c
        src/expr/exthdr.c
        src/expr/flow_offload.c
        src/expr/fib.c
        src/expr/fwd.c
        src/expr/last.c
        src/expr/limit.c
        src/expr/log.c
        src/expr/lookup.c
        src/expr/dynset.c
        src/expr/immediate.c
        src/expr/inner.c
        src/expr/match.c
        src/expr/meta.c
        src/expr/numgen.c
        src/expr/nat.c
        src/expr/tproxy.c
        src/expr/objref.c
        src/expr/payload.c
        src/expr/queue.c
        src/expr/quota.c
        src/expr/reject.c
        src/expr/rt.c
        src/expr/target.c
        src/expr/tunnel.c
        src/expr/masq.c
        src/expr/redir.c
        src/expr/hash.c
        src/expr/socket.c
        src/expr/synproxy.c
        src/expr/osf.c
        src/expr/xfrm.c
        src/obj/counter.c
        src/obj/ct_helper.c
        src/obj/quota.c
        src/obj/tunnel.c
        src/obj/limit.c
        src/obj/synproxy.c
        src/obj/ct_timeout.c
        src/obj/secmark.c
        src/obj/ct_expect.c
)

target_compile_options(nftnl 
    PRIVATE 
        $<$COMPILER_ID:GNU>:-pipe>
        $<$<COMPILE_LANGUAGE:C>:-Wall>
        $<$<COMPILE_LANGUAGE:C>:-Waggregate-return>
        $<$<COMPILE_LANGUAGE:C>:-Wmissing-declarations>
        $<$<COMPILE_LANGUAGE:C>:-Wmissing-prototypes>
        $<$<COMPILE_LANGUAGE:C>:-Wshadow>
        $<$<COMPILE_LANGUAGE:C>:-Wstrict-prototypes>
        $<$<COMPILE_LANGUAGE:C>:-Wformat=2>
        $<$<COMPILE_LANGUAGE:C>:-Wwrite-strings>
        $<$<COMPILE_LANGUAGE:CXX>:-D_FILE_OFFSET_BITS=64>
        $<$<COMPILE_LANGUAGE:CXX>:-D_REENTRANT>
        $<$<BOOL:${HAVE_VISIBILITY_HIDDEN}>:-fvisibility=hidden>
)

target_include_directories(nftnl 
    PUBLIC 
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<BUILD_INTERFACE:${CMAKE_BINARY_DIR}>
        $<INSTALL_INTERFACE:include>
)

target_link_libraries(nftnl PUBLIC ${LIBMNL_LIBRARIES})

target_link_options(nftnl PRIVATE
    -Wl,--version-script=${CMAKE_CURRENT_SOURCE_DIR}/src/libnftnl.map
    -version-info ${LIBVERSION}
)

# set target properties
set_target_properties(nftnl PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}

    C_STANDARD 99
    C_STANDARD_REQUIRED ON
    C_EXTENSIONS ON

    CXX_STANDARD 11
    CXX_STANDARD_REQUIRED ON
    CXX_EXTENSIONS ON

    LINK_DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/src/libnftnl.map
)


include(GNUInstallDirs)

# Install targets
install(TARGETS nftnl
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(DIRECTORY include/libnftnl DESTINATION include)

# Generate pkg-config file
configure_file(libnftnl.pc.in libnftnl.pc @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/libnftnl.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)

if(BUILD_EXAMPLES)
    # NOT IMPL
    message(FATAL_ERROR "Examples are not implemented yet.")
    add_subdirectory(examples)
endif()

if(BUILD_TESTS)
    # NOT IMPL
    message(FATAL_ERROR "Examples are not implemented yet.")
    enable_testing()
    add_subdirectory(tests)
endif()

if(BUILD_DOC)
    find_package(Doxygen)
    if(DOXYGEN_FOUND)
        configure_file(doxygen.cfg.in doxygen.cfg @ONLY)
        add_custom_target(doc ALL COMMAND ${DOXYGEN_EXECUTABLE} ${CMAKE_BINARY_DIR}/doxygen.cfg)
    endif()
endif()
