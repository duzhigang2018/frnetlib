cmake_minimum_required(VERSION 3.2)
project(frnetlib)
set(FRNETLIB_LINK_LIBRARIES "")

#Set module path
set( CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake_modules)

#User options
option(USE_SSL "Enable SSL support" OFF)
option(BUILD_EXAMPLES "Build frnetlib examples" ON)
option(BUILD_TESTS "Build frnetlib tests" ON)
option(BUILD_WEBSOCK "Enable WebSocket support" ON)
set(FRNETLIB_BUILD_SHARED_LIBS false CACHE BOOL "Build shared library.")
set(MAX_HTTP_HEADER_SIZE "0xC800" CACHE STRING "The maximum allowed HTTP header size in bytes")
set(MAX_HTTP_BODY_SIZE "0xA00000" CACHE STRING "The maximum allowed HTTP body size in bytes")
set(LISTEN_QUEUE_SIZE "64" CACHE STRING "The listen queue depth for fr::TcpListener/fr::SSLListener")
set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Build type" FORCE)

#Configure defines based on user options
add_definitions(-DMAX_HTTP_HEADER_SIZE=${MAX_HTTP_HEADER_SIZE})
add_definitions(-DMAX_HTTP_BODY_SIZE=${MAX_HTTP_BODY_SIZE})
add_definitions(-DLISTEN_QUEUE_SIZE=${LISTEN_QUEUE_SIZE})

if(USE_SSL)
    FIND_PACKAGE(MBEDTLS)
    INCLUDE_DIRECTORIES(${MBEDTLS_INCLUDE_DIR})
    set(SOURCE_FILES ${SOURCE_FILES} src/SSLSocket.cpp include/frnetlib/SSLSocket.h src/SSLListener.cpp include/frnetlib/SSLListener.h include/frnetlib/SSLContext.h)
    ADD_DEFINITIONS(-DUSE_SSL)
endif()

if(BUILD_WEBSOCK)
    set(SOURCE_FILES ${SOURCE_FILES} src/WebFrame.cpp include/frnetlib/WebFrame.h src/Sha1.cpp include/frnetlib/Sha1.h src/Base64.cpp include/frnetlib/Base64.h src/Sha1.cpp include/frnetlib/WebSocket.h)
endif()

add_definitions(-DNOMINMAX)
add_definitions(-Dhtonf)
add_definitions(-Dhtonll)
add_definitions(-Dntohll)
add_definitions(-Dhtond)
add_definitions(-Dntodh)

set( INCLUDE_PATH "${PROJECT_SOURCE_DIR}/include" )
set( SOURCE_PATH "${PROJECT_SOURCE_DIR}/src" )

set(SOURCE_FILES ${SOURCE_FILES} main.cpp src/TcpSocket.cpp include/frnetlib/TcpSocket.h src/TcpListener.cpp include/frnetlib/TcpListener.h src/Socket.cpp include/frnetlib/Socket.h include/frnetlib/Packet.h include/frnetlib/NetworkEncoding.h src/SocketSelector.cpp include/frnetlib/SocketSelector.h src/HttpRequest.cpp include/frnetlib/HttpRequest.h src/HttpResponse.cpp include/frnetlib/HttpResponse.h src/Http.cpp include/frnetlib/Http.h include/frnetlib/Packetable.h include/frnetlib/Listener.h src/URL.cpp include/frnetlib/URL.h include/frnetlib/Sendable.h include/frnetlib/version.h include/frnetlib/SocketDescriptor.h)

include_directories(include)
set(CORE_CXX_FLAGS "${CORE_CXX_FLAGS} -std=c++14 -Wall")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${CORE_CXX_FLAGS} -g -O0")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} ${CORE_CXX_FLAGS} -O2")

if(USE_SSL)
	set(FRNETLIB_LINK_LIBRARIES ${FRNETLIB_LINK_LIBRARIES} ${MBEDTLS_LIBRARIES})
endif()

if(NOT MSVC)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")
endif()

# Set the library output directory
set( LIBRARY_OUTPUT_PATH "${PROJECT_BINARY_DIR}/lib" )

# Add the library.
if( FRNETLIB_BUILD_SHARED_LIBS )
    add_library( frnetlib SHARED ${SOURCE_FILES} )
else()
    add_definitions( -DFRNETLIB_STATIC )
    add_library( frnetlib ${SOURCE_FILES} )
endif()

# Tell the compiler to export when necessary.
set_target_properties( frnetlib PROPERTIES DEFINE_SYMBOL FRNETLIB_EXPORTS )

if( WIN32 )
    set( SHARE_PATH "." )
    set( LIB_PATH "lib" )
elseif(APPLE)
    set( SHARE_PATH "${CMAKE_INSTALL_PREFIX}/share/FRNETLIB" )
    set( LIB_PATH "lib" )
else()
    set( SHARE_PATH "${CMAKE_INSTALL_PREFIX}/share/FRNETLIB" )

    if( LIB_SUFFIX )
        set( LIB_PATH "lib${LIB_SUFFIX}" )
    else()
        set( LIB_PATH "lib" )
    endif()
endif()

if(WIN32)
	set(FRNETLIB_LINK_LIBRARIES ${FRNETLIB_LINK_LIBRARIES} wsock32 ws2_32)
endif()

target_link_libraries(frnetlib ${FRNETLIB_LINK_LIBRARIES})

#Build Tests if needbe
if(BUILD_TESTS)
    set(EXT_PROJECTS_DIR ${PROJECT_SOURCE_DIR}/ext)
    add_subdirectory(${EXT_PROJECTS_DIR}/gtest)
    add_subdirectory(tests)
endif()

#Build examples if needbe
if(BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()

install(
        TARGETS frnetlib
        RUNTIME DESTINATION bin COMPONENT bin
        LIBRARY DESTINATION "${LIB_PATH}" COMPONENT bin
        ARCHIVE DESTINATION "${LIB_PATH}" COMPONENT dev
)

install(
        DIRECTORY include
        DESTINATION .
)
