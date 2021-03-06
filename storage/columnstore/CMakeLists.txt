#set(PLUGIN_COLUMNSTORE "NO" CACHE STRING "Enable ColumnStore engine")

if("NO" STREQUAL "${PLUGIN_COLUMNSTORE}")
  return()
endif()

# this does everything, gets the var from the correct scope, appends new
# values, sets in the correct scope
macro(APPEND_FOR_CPACK V)
  get_directory_property(var DIRECTORY columnstore DEFINITION ${V})
  set(${V} "${var}${ARGN}" PARENT_SCOPE)
endmacro()

IF(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64" OR
CMAKE_SYSTEM_PROCESSOR STREQUAL "amd64" OR
CMAKE_SYSTEM_PROCESSOR STREQUAL "i386" OR
CMAKE_SYSTEM_PROCESSOR STREQUAL "i686")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DCOLUMNSTORE_MATURITY=MariaDB_PLUGIN_MATURITY_BETA")
    add_subdirectory(columnstore)

    IF(TARGET columnstore)
        # Needed to bump the component changes up to the main scope
        APPEND_FOR_CPACK(CPACK_COMPONENTS_ALL)
        IF (RPM)
            APPEND_FOR_CPACK(CPACK_RPM_columnstore-engine_PACKAGE_REQUIRES " binutils jemalloc net-tools python3")
            APPEND_FOR_CPACK(CPACK_RPM_columnstore-engine_USER_FILELIST ";%ignore /var/lib;%ignore /var")
            APPEND_FOR_CPACK(CPACK_RPM_columnstore-engine_PACKAGE_CONFLICTS " thrift MariaDB-columnstore-platform MariaDB-columnstore-libs")
            # these three don't have the list semantics, so no append here
            SET(CPACK_RPM_columnstore-engine_PRE_INSTALL_SCRIPT_FILE ${CMAKE_CURRENT_SOURCE_DIR}/columnstore/build/preInstall_storage_engine.sh PARENT_SCOPE)
            SET(CPACK_RPM_columnstore-engine_POST_INSTALL_SCRIPT_FILE ${CMAKE_CURRENT_SOURCE_DIR}/columnstore/build/postInstall_storage_engine.sh PARENT_SCOPE)
            SET(CPACK_RPM_columnstore-engine_PRE_UNINSTALL_SCRIPT_FILE ${CMAKE_CURRENT_SOURCE_DIR}/columnstore/build/preUn_storage_engine.sh PARENT_SCOPE)
        ENDIF()
        INSTALL_MYSQL_TEST("${CMAKE_CURRENT_SOURCE_DIR}/mysql-test/" "plugin/columnstore")
    ENDIF()
ENDIF()
