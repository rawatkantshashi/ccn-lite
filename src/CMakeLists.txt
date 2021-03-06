cmake_minimum_required(VERSION 3.7)
project(ccn-lite NONE)

set(CMAKE_C_COMPILER_WORKS 1)
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)
set(CMAKE_SYSTEM_NAME Generic)

# CCNL flags
set(CCNL_BASIC_FLAGS
    -DUSE_DEBUG
    -DUSE_LOGGING
#    -DUSE_SIGNATURES
    -DUSE_DUP_CHECK
#    -DUSE_ECHO
    -DUSE_HMAC256
    -DUSE_STATS
    -DNEEDS_PACKET_CRAFTING
    -DNEEDS_PREFIX_MATCHING
    -DUSE_LINKLAYER
    CACHE PATH
    "basic build flags for CCN-lite"
)

add_definitions(${CCNL_BASIC_FLAGS})

if (NOT DEFINED CCNL_RIOT)
   set(CCNL_EXTRA_FLAGS 
        -DUSE_CCNxDIGEST
        -DUSE_MGMT
        -DUSE_UNIXSOCKET
        -DUSE_IPV4
        -DUSE_IPV6
        -DUSE_DEBUG_MALLOC
    )
    add_definitions(${CCNL_EXTRA_FLAGS})
endif()



# Platforms
set(CCNL_PLATFORM_FLAGS
    -DCCNL_UNIX
#    -DCCNL_ARDUINO
     CACHE PATH
    "platform flags for CCN-lite"
)
add_definitions(${CCNL_PLATFORM_FLAGS})

# Packet formats
set(CCNL_PACKETFORMAT_FLAGS
    -DUSE_SUITE_NDNTLV
    -DUSE_SUITE_CCNB
    -DUSE_SUITE_CCNTLV
    -DUSE_SUITE_CISTLV
    -DUSE_SUITE_IOTTLV
    -DUSE_SUITE_LOCALRPC
#    -DUSE_SUITE_COMPRESSED
    CACHE PATH
    "packet format flags for CCN-lite"
)
add_definitions(${CCNL_PACKETFORMAT_FLAGS})

# NFN flags
set(CCNL_NFN_FLAGS
    -DUSE_NFN
    -DUSE_NFN_REQUESTS
#    -DUSE_NFN_NSTRANS
    CACHE PATH
    "NFN flags for CCN-lite"
)

if(DEFINED CCNL_RIOT)
    set(CCNL_RIOT_FLAGS
        -DCCNL_APP_RX
        -DUSE_DUP_CHECK
        ${RIOT_CFLAGS}
         CACHE PATH
        "CCNL RIOT flags for CCN-lite"
    )
    add_definitions(${CCNL_RIOT_FLAGS})
endif()

if (NOT DEFINED CCNL_RIOT)
    add_definitions(${CCNL_NFN_FLAGS})
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wextra -Wall -Werror -std=c99 -g -pedantic")
else()
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wextra -Wall -Werror -std=c99 -g")
endif()

set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS} -g")

#add_subdirectory(ccnl-addons)
#if (DEFINED CCNL_RIOT)
  #  set(CMAKE_C_FLAGS ${RIOT_CFLAGS})
#endif()
if (NOT DEFINED CCNL_LINUXKERNEL)
    add_subdirectory(ccnl-core)
    add_subdirectory(ccnl-pkt)
    add_subdirectory(ccnl-fwd)
    if (NOT DEFINED CCNL_RIOT)
        add_subdirectory(ccnl-unix)
        add_subdirectory(ccnl-relay)
        add_subdirectory(ccnl-utils)
        add_subdirectory(ccnl-nfn)
        add_subdirectory(ccnl-pkt-compression)
    endif()
endif()
if (DEFINED CCNL_RIOT)
    add_subdirectory(ccnl-riot)
endif()
if (DEFINED CCNL_LINUXKERNEL)
    add_subdirectory(ccnl-lnxkernel)
endif()


if (NOT DEFINED CCNL_LINUXKERNEL)
if (NOT DEFINED CCNL_RIOT)
    add_dependencies(ccn-lite-relay ccnl-core ccnl-pkt ccnl-fwd ccnl-unix ccnl-nfn ccnl-pkt-compression)
endif()
endif()

find_package(Doxygen)
if (DOXYGEN_FOUND)
    set(DOXYGEN_IN ${CMAKE_CURRENT_SOURCE_DIR}/doc/Doxyfile.in)
    set(DOXYGEN_OUT ${CMAKE_CURRENT_BINARY_DIR}/doc/Doxyfile)
    configure_file(${DOXYGEN_IN} ${DOXYGEN_OUT} @ONLY)
    message("Doxygen build started")

    add_custom_target(doc
        COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_OUT}
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/doc
        COMMENT "Generating API documentation with Doxygen"
        VERBATIM )
endif()

# unused:
set(CCNL_DISABLED_FLAGS "USE_LINKLAYER USE_DEBUG USE_DEBUG_MALLOC USE_FRAG \
						USE_NACK USE_NFN USE_NFN_MONITOR USE_NFN_KEEPALIVE \
						USE_NFN_PULL USE_SCHEDULER USE_SIGNATURES -DUSE_HTTP_STATUS")
