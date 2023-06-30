cmake_minimum_required(VERSION 3.15)
project(ulog VERSION 1.0)

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

set(LIB_SOURCES
        ulog_read.c 
        ulog_write.c
        ulog_write_android.c
        ulog.cpp
)
if(UNIX)
  set(LIB_SOURCES
        ${LIB_SOURCES}
        ulog_write_android.c 
        ulog_write_bin.c 
        ulog_write_raw.c
  )
endif()


add_library(${PROJECT_NAME} SHARED ${LIB_SOURCES})

# checks if set up rpath exists for install
if(COMMAND set_up_rpath)
    set_up_rpath()
else()
    message("Set up rpath not defined!")
endif()

set(LIBULOG_HEADERS )
list(APPEND LIBULOG_HEADERS "${CMAKE_CURRENT_SOURCE_DIR}/include/ulog.h")
list(APPEND LIBULOG_HEADERS "${CMAKE_CURRENT_SOURCE_DIR}/include/ulograw.h")

#option(BUILD_SHARED_LIBS "Build using shared libraries" ON)
set_target_properties(${PROJECT_NAME} PROPERTIES
                    POSITION_INDEPENDENT_CODE ${BUILD_SHARED_LIBS}
)
set_target_properties(${PROJECT_NAME} PROPERTIES
                    PUBLIC_HEADER "${LIBULOG_HEADERS}"
)

target_include_directories(${PROJECT_NAME} PUBLIC
                            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                            $<INSTALL_INTERFACE:include>
)

target_compile_definitions(${PROJECT_NAME} PRIVATE "LIBULOG_HEADERS=${CMAKE_CURRENT_SOURCE_DIR}/include/ulog.h:${CMAKE_CURRENT_SOURCE_DIR}/include/ulograw.h;")
target_compile_options(${PROJECT_NAME} PRIVATE "-fvisibility=hidden")


target_link_libraries(${PROJECT_NAME}
                    PRIVATE
                      pthread
)

install(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}-targets
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR} ${CMAKE_INSTALL_INCLUDEDIR}/libulog
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(EXPORT ${PROJECT_NAME}-targets
    FILE ${PROJECT_NAME}-targets.cmake
    NAMESPACE ${PROJECT_NAME}::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in"
    ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-config.cmake
    INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}"
)

install(
    FILES ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-config.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

install(FILES ${LIBULOG_HEADERS} DESTINATION ${CMAKE_INSTALL_FULL_INCLUDEDIR}/libulog)
