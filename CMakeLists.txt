cmake_minimum_required(VERSION 3.0)

include(${CMAKE_SOURCE_DIR}/system-config.cmake OPTIONAL)

project(bacs)

if(BunsanCMakeBuiltin)
    BunsanCMakeLoad()
else()
    find_package(BunsanCMake CONFIG REQUIRED)
endif()

bunsan_add_shared_library(${PROJECT_NAME}
    src/lib/helper.cpp
    src/lib/test.cpp
    src/lib/db.cpp
    src/lib/run.cpp
    src/lib/tempfiles.cpp
    src/lib/log.cpp
    src/lib/util.cpp
    src/lib/common.cpp
    src/lib/problem.cpp
    src/lib/config.cpp
    src/lib/submit.cpp
)
bunsan_use_bunsan_package(${PROJECT_NAME} bunsan_pm bunsan_pm_compatibility)

# MySQL
find_path(mysql_include_dir NAMES mysql.h PATHS /usr/include /usr/include/mysql /usr/local/include /usr/local/include/mysql)
find_path(mysql_link_dir NAMES libmysqlclient.so PATHS /usr/lib /usr/lib/mysql /usr/local/lib /usr/local/lib/mysql)
link_directories(${PROJECT_NAME} PRIVATE ${mysql_link_dir})
message(STATUS "mysql.h was found in ${mysql_include_dir}")
message(STATUS "libmysqlclient.so was found in ${mysql_include_dir}")
target_include_directories(${PROJECT_NAME} PUBLIC ${mysql_include_dir})
target_link_libraries(${PROJECT_NAME} PUBLIC mysqlclient)

bunsan_use_boost(${PROJECT_NAME} system filesystem)

bunsan_add_executable(${PROJECT_NAME}d src/bin/bacsd.cpp)
bunsan_use_target(${PROJECT_NAME}d ${PROJECT_NAME})

bunsan_add_executable(checkdef src/bin/checkdef.cpp)
bunsan_add_executable(limit_run src/bin/limit_run.cpp)
target_link_libraries(limit_run rt)
bunsan_add_executable(java_compile src/bin/java_compile.c)
bunsan_add_executable(wipe src/bin/wipe.c)
bunsan_add_executable(java_run src/bin/java_run.c)

macro(bacs_install_utils)
    bunsan_install_targets_prepare(${ARGN})
    install(TARGETS ${ARGN}
            DESTINATION ${CMAKE_INSTALL_LIBDIR}/bacs/legacy)
endmacro()

bunsan_install_headers()
bunsan_install_targets(
    ${PROJECT_NAME}
    ${PROJECT_NAME}d
)
bacs_install_utils(
    checkdef
    limit_run
    java_compile
    wipe
    java_run
)
install(PROGRAMS src/bin/py3_compile.py
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/bacs/legacy)
file(COPY src/bin/py3_compile.py
     DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
install(PROGRAMS init_env.sh
        DESTINATION ${CMAKE_INSTALL_BINDIR}
        RENAME bacs-legacy-makeenv)
bunsan_install_project()
