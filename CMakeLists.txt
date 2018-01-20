# MIT License
#
# Copyright (c) 2018 Joerg Hallmann
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.
#
# https://github.com/joerghall/cbt-devtool-gcc
#
cmake_minimum_required(VERSION 3.5)

if (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/_submodules/cbt/cmake/setup.cmake)
    include(${CMAKE_CURRENT_SOURCE_DIR}/_submodules/cbt/cmake/setup.cmake)
else ()
    message(FATAL_ERROR "Unable to find buildtools, please check if the submodule is initialized")
endif ()

if(BUILD_WINDOWS)
    add_artifact("python2" "https://h1grid.com/artifactory/cbt/devtools/python2/2.7.14.43.rcd588a99/python2-2.7.14.43.rcd588a99-windows.tgz" ${ARTIFACT_TOOLSETS_CACHE} "python2/2.7.14.43.rcd588a99/windows" "python2-2.7.14.43.rcd588a99-windows.tgz")
#    add_artifact("ninja" "https://h1grid.com:443/artifactory/cbt/devtools/ninja/1.8.2/ninja-1.8.2-windows.tgz" ${ARTIFACT_TOOLSETS_CACHE} "ninja/1.8.2/windows" "ninja-1.8.2-windows.tgz")
#    add_artifact("pigz" "https://h1grid.com:443/artifactory/cbt/devtools/pigz/2.3.1/pigz-2.3.1-windows.tgz" ${ARTIFACT_TOOLSETS_CACHE} "pigz/2.3.1/windows" "pigz-2.3.1-windows.tgz")
elseif(BUILD_OSX)
    add_artifact("python2" "https://h1grid.com/artifactory/cbt/devtools/python2/2.7.14.43.rcd588a99/python2-2.7.14.43.rcd588a99-osx.tgz" ${ARTIFACT_TOOLSETS_CACHE} "python2/2.7.14.43.rcd588a99/osx" "python2-2.7.14.43.rcd588a99-osx.tgz")
#    add_artifact("ninja" "https://h1grid.com:443/artifactory/cbt/devtools/ninja/1.8.2/ninja-1.8.2-osx.tgz" ${ARTIFACT_TOOLSETS_CACHE} "ninja/1.8.2/osx" "ninja-1.8.2-osx.tgz")
#    add_artifact("pigz" "https://h1grid.com:443/artifactory/cbt/devtools/pigz/2.3.4/pigz-2.3.4-osx.tgz" ${ARTIFACT_TOOLSETS_CACHE} "pigz/2.3.4/osx" "pigz-2.3.4-osx.tgz")
elseif(BUILD_LINUX)
    add_artifact("python2" "https://h1grid.com/artifactory/cbt/devtools/python2/2.7.14.43.rcd588a99/python2-2.7.14.43.rcd588a99-linux.tgz" ${ARTIFACT_TOOLSETS_CACHE} "python2/2.7.14.43.rcd588a99/linux" "python2-2.7.14.43.rcd588a99-linux.tgz")
#    add_artifact("ninja" "https://h1grid.com:443/artifactory/cbt/devtools/ninja/1.8.2/ninja-1.8.2-linux.tgz" ${ARTIFACT_TOOLSETS_CACHE} "ninja/1.8.2/linux" "ninja-1.8.2-windows.tgz")
#    add_artifact("pigz" "https://h1grid.com:443/artifactory/cbt/devtools/pigz/2.3.1/pigz-2.3.4-linux.tgz" ${ARTIFACT_TOOLSETS_CACHE} "pigz/2.3.4/linux" "pigz-2.3.4-windows.tgz")
endif()

include(python2)

set(CMAKE_CXX_STANDARD 14)

project(cbt-devtool-gcc)

enable_testing()

file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CPACK_OUTPUT_FILE_PREFIX ${CMAKE_BINARY_DIR}/packages)

# Standard locations
#
set(SOURCE_UPLOAD_DIR ${CMAKE_BINARY_DIR}/upload)
set(SOURCE_DOWNLOAD_DIR ${CMAKE_BINARY_DIR}/download)
set(SOURCE_INSTALL_DIR ${CMAKE_BINARY_DIR}/install)
set(SOURCE_SRC_DIR ${CMAKE_BINARY_DIR}/src)
set(TARGET_INSTALL_DIR ${CMAKE_BINARY_DIR}/build)

# Generate version information for package
#
execute_process(COMMAND git rev-list HEAD
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    RESULT_VARIABLE VERSION_RESULT
    OUTPUT_VARIABLE VERSION_DATA
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

string(REPLACE "\n" ";" VERSION_DATA ${VERSION_DATA} )
list(LENGTH VERSION_DATA VERSION_REVISION)
list(GET VERSION_DATA 0 VERSION_HASH)
string(SUBSTRING ${VERSION_HASH} 0 8 VERSION_SHORTHASH)

# Python specifics
#
set(NINJA_VERSION 1.8.2)
set(NINJA_VERSION_PACKAGE ${NINJA_VERSION}.${VERSION_REVISION}.r${VERSION_SHORTHASH})

MESSAGE(STATUS "Building package version ninja-${NINJA_VERSION_PACKAGE}.*.tgz")

set(PYTHON_SOURCE_SOURCE_URL_WINDOWS https://github.com/ninja-build/ninja/releases/download/v${NINJA_VERSION}/ninja-win.zip)
set(PYTHON_SOURCE_SOURCE_URL_OSX https://github.com/ninja-build/ninja/releases/download/v${NINJA_VERSION}/ninja-mac.zip)
set(PYTHON_SOURCE_SOURCE_URL_LINUX https://github.com/ninja-build/ninja/releases/download/v${NINJA_VERSION}/ninja-linux.zip)
set(PYTHON_SOURCE_TARGET_FILE_WINDOWS ${SOURCE_DOWNLOAD_DIR}/ninja-win.zip)
set(PYTHON_SOURCE_TARGET_FILE_OSX ${SOURCE_DOWNLOAD_DIR}/ninja-mac.zip)
set(PYTHON_SOURCE_TARGET_FILE_LINUX ${SOURCE_DOWNLOAD_DIR}/ninja-linux.zip)

add_custom_target(download_ninja
    COMMAND ${CMAKE_COMMAND} -E make_directory ${SOURCE_DOWNLOAD_DIR}
    COMMAND ${CMAKE_COMMAND} -DSOURCE=${PYTHON_SOURCE_SOURCE_URL_WINDOWS} -DTARGET=${PYTHON_SOURCE_TARGET_FILE_WINDOWS} -P ${download_COMMAND}
    COMMAND ${CMAKE_COMMAND} -DSOURCE=${PYTHON_SOURCE_SOURCE_URL_OSX} -DTARGET=${PYTHON_SOURCE_TARGET_FILE_OSX} -P ${download_COMMAND}
    COMMAND ${CMAKE_COMMAND} -DSOURCE=${PYTHON_SOURCE_SOURCE_URL_LINUX} -DTARGET=${PYTHON_SOURCE_TARGET_FILE_LINUX} -P ${download_COMMAND}
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
)

add_custom_target(unpack_ninja
    COMMAND ${CMAKE_COMMAND} -E make_directory ${SOURCE_SRC_DIR}/windows/ninja/${NINJA_VERSION}/windows
    COMMAND ${CMAKE_COMMAND} -E make_directory ${SOURCE_SRC_DIR}/linux/ninja/${NINJA_VERSION}/linux
    COMMAND ${CMAKE_COMMAND} -E make_directory ${SOURCE_SRC_DIR}/osx/ninja/${NINJA_VERSION}/osx
    COMMAND python2 ${unzip_py_COMMAND} --file ${PYTHON_SOURCE_TARGET_FILE_WINDOWS} --target ${SOURCE_SRC_DIR}/windows/ninja/${NINJA_VERSION}/windows
    COMMAND python2 ${unzip_py_COMMAND} --file ${PYTHON_SOURCE_TARGET_FILE_LINUX} --target ${SOURCE_SRC_DIR}/linux/ninja/${NINJA_VERSION}/linux
    COMMAND python2 ${unzip_py_COMMAND} --file ${PYTHON_SOURCE_TARGET_FILE_OSX} --target ${SOURCE_SRC_DIR}/osx/ninja/${NINJA_VERSION}/osx
    DEPENDS download_ninja
)

add_custom_target(package_ninja
    COMMAND ${CMAKE_COMMAND} -E make_directory ${SOURCE_UPLOAD_DIR}
    COMMAND ${CMAKE_COMMAND} -E tar czf ${SOURCE_UPLOAD_DIR}/ninja-${NINJA_VERSION_PACKAGE}-windows.tgz windows/ninja
    COMMAND ${CMAKE_COMMAND} -E tar czf ${SOURCE_UPLOAD_DIR}/ninja-${NINJA_VERSION_PACKAGE}-linux.tgz linux/ninja
    COMMAND ${CMAKE_COMMAND} -E tar czf ${SOURCE_UPLOAD_DIR}/ninja-${NINJA_VERSION_PACKAGE}-osx.tgz osx/ninja
    WORKING_DIRECTORY ${SOURCE_SRC_DIR}
    DEPENDS unpack_ninja
)

add_custom_target(publish_ninja
    COMMAND ${python2_BIN} ${upload_py_COMMAND} --file ${SOURCE_UPLOAD_DIR}/ninja-${NINJA_VERSION_PACKAGE}-windows.tgz --url https://h1grid.com:443/artifactory/cbt/devtools/ninja/${NINJA_VERSION_PACKAGE}/ninja-${NINJA_VERSION_PACKAGE}-windows.tgz
    COMMAND ${python2_BIN} ${upload_py_COMMAND} --file ${SOURCE_UPLOAD_DIR}/ninja-${NINJA_VERSION_PACKAGE}-linux.tgz --url https://h1grid.com:443/artifactory/cbt/devtools/ninja/${NINJA_VERSION_PACKAGE}/ninja-${NINJA_VERSION_PACKAGE}-linux.tgz
    COMMAND ${python2_BIN} ${upload_py_COMMAND} --file ${SOURCE_UPLOAD_DIR}/ninja-${NINJA_VERSION_PACKAGE}-osx.tgz --url https://h1grid.com:443/artifactory/cbt/devtools/ninja/${NINJA_VERSION_PACKAGE}/ninja-${NINJA_VERSION_PACKAGE}-osx.tgz
    DEPENDS package_ninja
)
