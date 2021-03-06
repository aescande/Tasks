# Copyright 2012-2016 CNRS-UM LIRMM, CNRS-AIST JRL
#
# This file is part of Tasks.
#
# Tasks is free software: you can redistribute it and/or modify
# it under the terms of the GNU Lesser General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# Tasks is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public License
# along with Tasks.  If not, see <http://www.gnu.org/licenses/>.

INCLUDE(../../cmake/python.cmake)

# Look for Python 2.7
SET(Python_ADDITIONAL_VERSIONS 2.7)
FINDPYTHON()

# Create the package in build dir for testing purpose
SET(LIBRARY_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR}/tasks)
CONFIGURE_FILE(__init__.py ${CMAKE_CURRENT_BINARY_DIR}/tasks/__init__.py COPYONLY)
SET(OUTPUT_BINDING ${CMAKE_CURRENT_BINARY_DIR}/tasks.cpp)

# Generate Python binding code
ADD_CUSTOM_TARGET(generate_python_bindings
  COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/generate.py ${OUTPUT_BINDING}
  DEPENDS generate.py
)

SET_SOURCE_FILES_PROPERTIES(${OUTPUT_BINDING} PROPERTIES GENERATED 1)

# Build the library
SET(SOURCES ${OUTPUT_BINDING})

INCLUDE_DIRECTORIES(SYSTEM ${PYTHON_INCLUDE_DIRS})
INCLUDE_DIRECTORIES(.)
INCLUDE_DIRECTORIES(../../src)

ADD_LIBRARY(_tasks SHARED ${SOURCES})
ADD_DEPENDENCIES(_tasks generate_python_bindings)

PKG_CONFIG_USE_DEPENDENCY(_tasks sch-core)
PKG_CONFIG_USE_DEPENDENCY(_tasks SpaceVecAlg)
PKG_CONFIG_USE_DEPENDENCY(_tasks RBDyn)
PKG_CONFIG_USE_DEPENDENCY(_tasks eigen-qld)
if(${EIGEN_LSSOL_FOUND})
  PKG_CONFIG_USE_DEPENDENCY(_tasks eigen-lssol)
endif()

TARGET_LINK_LIBRARIES(_tasks Tasks)
SET_TARGET_PROPERTIES(_tasks PROPERTIES PREFIX "")

# Install rules
INSTALL(TARGETS _tasks DESTINATION "${PYTHON_SITELIB}/tasks")
PYTHON_INSTALL_BUILD(tasks __init__.py "${PYTHON_SITELIB}")
