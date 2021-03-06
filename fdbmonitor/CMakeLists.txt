set(FDBMONITOR_SRCS ConvertUTF.h SimpleIni.h fdbmonitor.cpp)

add_executable(fdbmonitor ${FDBMONITOR_SRCS})
strip_debug_symbols(fdbmonitor)
assert_no_version_h(fdbmonitor)
if(UNIX AND NOT APPLE)
    target_link_libraries(fdbmonitor rt)
endif()
# FIXME: This include directory is an ugly hack. We probably want to fix this
# as soon as we get rid of the old build system
target_include_directories(fdbmonitor PRIVATE ${CMAKE_BINARY_DIR}/fdbclient)

fdb_install(TARGETS fdbmonitor DESTINATION fdbmonitor COMPONENT server)

# Create a local sandbox for quick manual testing without simulator
file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/sandbox/data)
file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/sandbox/logs)
configure_file(${CMAKE_SOURCE_DIR}/cmake/Sandbox.conf.cmake
  ${CMAKE_BINARY_DIR}/sandbox/foundationdb.conf)

# this is not portable on Windows - but fdbmonitor isn't built there anyways...
add_custom_target(clean_sandbox
  COMMAND rm -rf ${CMAKE_BINARY_DIR}/sandbox/data/* && rm -rf ${CMAKE_BINARY_DIR}/sandbox/logs/*
  COMMENT "Cleaning existing sandbox")

add_custom_target(start_sandbox
  COMMAND ${CMAKE_BINARY_DIR}/bin/fdbmonitor --conffile ${CMAKE_BINARY_DIR}/sandbox/foundationdb.conf
                                             --lockfile ${CMAKE_BINARY_DIR}/sandbox/fdbmonitor.lock)

add_dependencies(start_sandbox fdbmonitor fdbserver)
