# Copyright (c) Microsoft Corporation.
# Licensed under the MIT License.

add_library(logging_inc INTERFACE)
target_link_libraries(inc INTERFACE logging_inc)

if(QUIC_ENABLE_LOGGING)
    if(WIN32)
        target_link_libraries(inc INTERFACE MsQuicEtw_Header)
        add_library(logging INTERFACE)
    else()
        target_include_directories(logging_inc INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/common)
        target_include_directories(logging_inc INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/${QUIC_LTTNG_PLATFORM})
        target_include_directories(logging_inc INTERFACE ${LTTNGUST_INCLUDE_DIRS})

        FILE(GLOB LOGGING_FILES ${CMAKE_CURRENT_SOURCE_DIR}/${QUIC_LTTNG_PLATFORM}/*.c)
        add_library(logging STATIC ${LOGGING_FILES})
        target_link_libraries(logging PRIVATE inc)

        add_library(msquic.lttng SHARED ${LOGGING_FILES})
        target_compile_definitions(msquic.lttng PRIVATE BUILDING_TRACEPOINT_PROVIDER)
        target_link_libraries(msquic.lttng PRIVATE logging_inc inc)
        target_link_libraries(msquic.lttng PRIVATE ${LTTNGUST_LIBRARIES})

        install(TARGETS msquic.lttng DESTINATION lib)
    endif()
else()
    add_library(logging INTERFACE)
endif()
