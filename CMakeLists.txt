cmake_minimum_required(VERSION 3.9 FATAL_ERROR)
project(IncludeSpdlog  VERSION 0.0.1  DESCRIPTION IncludeSpdlog  LANGUAGES C CXX)

set(TARGET_NAME IncludeSpdlog)
set(DEFAULT_BUILD_TYPE "Release")

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${FLAGS_COMMON}")


if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${DEFAULT_BUILD_TYPE}' as none was specified.")
  set(CMAKE_BUILD_TYPE "${DEFAULT_BUILD_TYPE}" CACHE STRING "Choose the type of build." FORCE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()


enable_testing()

macro(clone_build libname)
    message(STATUS "Clone_build: benchmarks ${libname}") 
	configure_file(clone_build/${libname}.in ${libname}-download/CMakeLists.txt)
	execute_process(COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}" .
    WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/${libname}-download")
	execute_process(COMMAND "${CMAKE_COMMAND}" --build .
	WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/${libname}-download")
	add_subdirectory("${CMAKE_BINARY_DIR}/${libname}-src"
                 "${CMAKE_BINARY_DIR}/${libname}-build")

    include_directories("${libname}-src/include")
	#install(EXPORT ${libname} DESTINATION share/${libname}/cmake)
	install(TARGETS ${libname} EXPORT ${libname}Config
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
	install(TARGETS ${libname} DESTINATION lib)
	#install(FILES  ${HEADERS} DESTINATION include)
	install(EXPORT ${libname}Config DESTINATION share/${libname}/cmake)

	export(TARGETS ${libname} FILE ${libname}Config.cmake)

endmacro()

clone_build(spdlog)

set(SRC src/includespdlog.cc)

set(HEADERS include/includespdlog.h)

add_library(${PROJECT_NAME} STATIC
    ${SRC}
    ${HEADERS}
)

target_include_directories(${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
    PRIVATE src
)

target_link_libraries(${PROJECT_NAME} spdlog::spdlog)

if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set (CMAKE_INSTALL_PREFIX "/usr" CACHE PATH "default install path" FORCE )
endif()

install(TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME}Config
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(TARGETS ${PROJECT_NAME} DESTINATION lib)
install(FILES  ${HEADERS} DESTINATION include)
install(EXPORT ${PROJECT_NAME}Config DESTINATION share/${PROJECT_NAME}/cmake)

export(TARGETS ${PROJECT_NAME} FILE ${PROJECT_NAME}Config.cmake)
