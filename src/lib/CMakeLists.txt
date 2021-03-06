# Create object libraries, then create master library
# http://www.cmake.org/Wiki/CMake/Tutorials/Object_Library

include(sugar_include)
sugar_include(.)

list(APPEND GPUOPENCV_PUBLIC_HDRS 
  ${GPUOPENCV_COMPLEX_PUBLIC_HDRS} 
  ${GPUOPENCV_SIMPLE_PUBLIC_HDRS}
)

#
# Build and install a single library or framework from our set of object libraries
#

add_library(simple OBJECT ${GPUOPENCV_SIMPLE_SRCS})
add_library(complex OBJECT ${GPUOPENCV_COMPLEX_SRCS})
add_library(GPUOpenCV ${GPUOPENCV_SRCS} $<TARGET_OBJECTS:simple> $<TARGET_OBJECTS:complex>)

include(GenerateExportHeader)
generate_export_header(GPUOpenCV)

target_include_directories(GPUOpenCV PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>")

# Note: this will be handled automatically in case simple/complex is not an OBJECT libraries
target_include_directories(simple PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>")
target_include_directories(complex PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>")

target_link_libraries(GPUOpenCV PRIVATE 
  ${OpenCV_LIBS} 
  ${TEST_SCOPE_ISSUE_LIBS} # Conditionally defined
  )

if(APPLE)
  target_link_libraries(
      GPUOpenCV
      PRIVATE
      "-Wl,-exported_symbols_list"
      "${CMAKE_CURRENT_LIST_DIR}/../../exports/apple"
  )
endif()

set_target_properties(GPUOpenCV PROPERTIES
  CXX_VISIBILITY_PRESET hidden  ### HIDE
  VISIBILITY_INLINES_HIDDEN ON
  XCODE_ATTRIBUTE_GCC_INLINES_ARE_PRIVATE_EXTERN YES 
  XCODE_ATTRIBUTE_GCC_SYMBOLS_PRIVATE_EXTERN YES
  VERSION "${GPUOpenCV_VERSION}"
  SOVERSION "${GPUOpenCV_VERSION_MAJOR}"
  PUBLIC_HEADER "${GPUOPENCV_PUBLIC_HDRS}" 
  INSTALL_RPATH_USE_LINK_PATH TRUE
)

install(TARGETS GPUOpenCV
  FRAMEWORK DESTINATION ${FRAMEWORK_INSTALL_DIR}
  LIBRARY DESTINATION ${LIB_INSTALL_DIR}
  RUNTIME DESTINATION ${BIN_INSTALL_DIR}
  ARCHIVE DESTINATION ${LIB_INSTALL_DIR}
  PUBLIC_HEADER DESTINATION ${INCLUDE_INSTALL_DIR}/GPUOpenCV
)

###################
##### install #####
###################

message("========== GPUOPENCV_PUBLIC_HDRS ${GPUOPENCV_PUBLIC_HDRS} ==============")

set(CMAKE_PROJECT_NAME GPUOpenCV) # overwrite drishtisdk

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(include_install_dir "include")
set(config_install_dir "share/${CMAKE_PROJECT_NAME}")
set(version_config "${generated_dir}/${CMAKE_PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${CMAKE_PROJECT_NAME}Config.cmake")
set(targets_export_name "${CMAKE_PROJECT_NAME}Targets")
set(namespace "${CMAKE_PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file("${version_config}" VERSION "${GPUOpenCV_VERSION}" COMPATIBILITY SameMajorVersion)

# Note: variable 'targets_export_name' used
configure_file("${CMAKE_SOURCE_DIR}/cmake/Config.cmake.in" "${project_config}" @ONLY)

#set(CMAKE_INSTALL_RPATH $LIBRARY_OUTPUT_PATH)

install(
  TARGETS GPUOpenCV
  EXPORT "${targets_export_name}"
  LIBRARY DESTINATION "${LIB_INSTALL_DIR}"
  ARCHIVE DESTINATION "${LIB_INSTALL_DIR}"
  RUNTIME DESTINATION "${BIN_INSTALL_DIR}"
  INCLUDES DESTINATION "${include_install_dir}"
  )

install(
  FILES 
  ${GPUOPENCV_PUBLIC_HDRS}
  DESTINATION "${include_install_dir}/${CMAKE_PROJECT_NAME}"
  )

install(
  FILES
  "${CMAKE_CURRENT_BINARY_DIR}/gpuopencv_export.h"
  DESTINATION "${include_install_dir}"
  )

install(
  FILES "${project_config}" "${version_config}"
  DESTINATION "${config_install_dir}"
  )

install(
  EXPORT "${targets_export_name}"
  NAMESPACE "${namespace}"
  DESTINATION "${config_install_dir}"
  )

##
## Get extra dependencies
##

# http://stackoverflow.com/a/24389006

# include(GetPrerequisites)
# get_prerequisites("${TARGET_SDK}" DEPENDENCIES 0 0 "" "")

# foreach(DEPENDENCY_FILE ${DEPENDENCIES})
#   gp_resolve_item("${TARGET_SDK}" "${DEPENDENCY_FILE}" "" "" resolved_file)
#   message("resolved_file='${resolved_file}'")
# endforeach()
