# SPDX-FileCopyrightText: 2015-2023 Greenbone AG
#
# SPDX-License-Identifier: GPL-2.0-or-later

## Library

include(FindPkgConfig)

if(NOT PKG_CONFIG_FOUND)
  message(FATAL_ERROR "pkg-config executable not found. Aborting.")
endif(NOT PKG_CONFIG_FOUND)

## Dependency checks

pkg_check_modules(GLIB REQUIRED glib-2.0>=2.42)

include_directories(${GLIB_INCLUDE_DIRS})

set(FILES osp.c)

set(HEADERS osp.h)

if(BUILD_STATIC)
  add_library(gvm_osp_static STATIC ${FILES})
  set_target_properties(gvm_osp_static PROPERTIES OUTPUT_NAME "gvm_osp")
  set_target_properties(gvm_osp_static PROPERTIES CLEAN_DIRECT_OUTPUT 1)
  set_target_properties(gvm_osp_static PROPERTIES PUBLIC_HEADER "${HEADERS}")
endif(BUILD_STATIC)

if(BUILD_SHARED)
  add_library(gvm_osp_shared SHARED ${FILES})
  set_target_properties(gvm_osp_shared PROPERTIES OUTPUT_NAME "gvm_osp")
  set_target_properties(gvm_osp_shared PROPERTIES CLEAN_DIRECT_OUTPUT 1)
  set_target_properties(
    gvm_osp_shared
    PROPERTIES SOVERSION "${PROJECT_VERSION_MAJOR}"
  )
  set_target_properties(
    gvm_osp_shared
    PROPERTIES VERSION "${CPACK_PACKAGE_VERSION}"
  )
  set_target_properties(gvm_osp_shared PROPERTIES PUBLIC_HEADER "${HEADERS}")

  target_link_libraries(
    gvm_osp_shared
    LINK_PRIVATE gvm_util_shared ${GLIB_LDFLAGS} ${LINKER_HARDENING_FLAGS}
  )
endif(BUILD_SHARED)

## Tests

if(BUILD_TESTS)
  add_unit_test(
    osp-test
    osp_tests.c
    gvm_base_shared
    gvm_util_shared
    ${GLIB_LDFLAGS}
    ${LINKER_HARDENING_FLAGS}
  )
endif(BUILD_TESTS)

## Install
configure_file(libgvm_osp.pc.in ${CMAKE_BINARY_DIR}/libgvm_osp.pc @ONLY)

install(FILES ${CMAKE_BINARY_DIR}/libgvm_osp.pc DESTINATION ${LIBDIR}/pkgconfig)

if(BUILD_STATIC)
  install(
    TARGETS gvm_osp_static
    RUNTIME DESTINATION ${BINDIR}
    ARCHIVE DESTINATION ${LIBDIR}
    PUBLIC_HEADER DESTINATION "${INCLUDEDIR}/gvm/osp"
  )
endif(BUILD_STATIC)
if(BUILD_SHARED)
  install(
    TARGETS gvm_osp_shared
    RUNTIME DESTINATION ${BINDIR}
    LIBRARY DESTINATION ${LIBDIR}
    ARCHIVE DESTINATION ${LIBDIR}
    PUBLIC_HEADER DESTINATION "${INCLUDEDIR}/gvm/osp"
  )
endif(BUILD_SHARED)

## End
