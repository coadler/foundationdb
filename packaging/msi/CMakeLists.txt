find_package(WIX)

if(WIX_FOUND)
  add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/FDBInstaller.wxs
    COMMAND ${CMAKE_COMMAND}
            -DOUT=${CMAKE_CURRENT_BINARY_DIR}/FDBInstaller.wxs
            -DIN=${CMAKE_CURRENT_SOURCE_DIR}/FDBInstaller.wxs.cmake
            -Dfdbserver=$<TARGET_FILE:fdbserver>
            -Dfdbcli=$<TARGET_FILE:fdbcli>
            -Dfdbbackup=$<TARGET_FILE:fdbbackup>
            -Dfdbmonitor=$<TARGET_FILE:fdbmonitor>
            -Dfdbc=$<TARGET_FILE:fdb_c>
            -DSOURCE_ROOT=${CMAKE_SOURCE_DIR}
            -DBUILD_ROOT=${CMAKE_BINARY_DIR}
            -DCURRENT_SOURCE_DIR=${CMAKE_CURRENT_SOURCE_DIR}
            -DVERSION=${CMAKE_PROJECT_VERSION}
            -DVERSION_NAME=${FDB_VERSION}
            -P ${CMAKE_CURRENT_SOURCE_DIR}/generate_wxs.cmake
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/FDBInstaller.wxs ${CMAKE_CURRENT_SOURCE_DIR}/generate_wsx.cmake
    COMMENT "Generate WIX file")
  add_custom_target(wix_file DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/FDBInstaller.wxs)
  add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/FDBInstaller.wixobj
    COMMAND ${WIX_CANDLE} FDBInstaller.wxs
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    DEPENDS wix_file
    COMMENT "Create WIX Object file"
    )
  add_custom_target(wixobj DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/FDBInstaller.wixobj)
  add_dependencies(wixobj fdbserver fdbcli fdbmonitor fdb_c fdbbackup python_binding)
  string(REPLACE "/" "\\" msi_out_file "${CMAKE_BINARY_DIR}/packages/FDBInstaller.msi")
  add_custom_command(
    OUTPUT ${CMAKE_BINARY_DIR}/packages/FDBInstaller.msi
    COMMAND ${WIX_LIGHT} FDBInstaller.wixobj -ext WixUIExtension -out ${msi_out_file}
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    COMMENT "Generate MSI installer"
    )
  add_custom_target(installer DEPENDS ${CMAKE_BINARY_DIR}/packages/FDBInstaller.msi)
  add_dependencies(installer wixobj)
  add_dependencies(packages installer)
  #add_dependencies(installer fdbserver fdbcli fdbmonitor fdbbackup)
else()
  message(WARNING "Could NOT find WIX - will not build installer")
endif()
