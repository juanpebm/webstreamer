#-------------------------------------------------------------------------------
# web streamer
#
# Copyright (c) 2017 RWTH Aachen University, Germany,
# Virtual Reality & Immersive Visualization Group.
#-------------------------------------------------------------------------------
#                                 License
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.
#-------------------------------------------------------------------------------

file(GLOB WEBSTREAMER_SOURCES src/*.cpp)
file(GLOB WEBSTREAMER_HEADERS src/*.hpp)
file(GLOB WEBSTREAMER_API_HEADERS include/webstreamer/*.hpp)

add_library(webstreamer
  ${WEBSTREAMER_SOURCES}
  ${WEBSTREAMER_HEADERS}
  ${WEBSTREAMER_API_HEADERS}
)

add_test(NAME "cpplint@webstreamer" COMMAND "python" "${CMAKE_SOURCE_DIR}/cpplint.py"
  ${WEBSTREAMER_SOURCES}
  ${WEBSTREAMER_HEADERS}
  ${WEBSTREAMER_API_HEADERS}
)

generate_export_header(
  webstreamer
  EXPORT_FILE_NAME "${CMAKE_CURRENT_BINARY_DIR}/include/webstreamer/export.hpp"
)

target_include_directories(
  webstreamer
  PUBLIC ${CMAKE_CURRENT_BINARY_DIR}/include
  PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include
)

set_warning_levels_rwth(webstreamer)


# --- dependencies ---
# Poco C++ libraries
find_package(Poco COMPONENTS Util Net REQUIRED)
target_link_libraries(webstreamer
  PUBLIC Poco::JSON
  PUBLIC Poco::Net
  PUBLIC Poco::Util
)

# x264
find_package(x264 REQUIRED)
target_include_directories(webstreamer PUBLIC ${X264_INCLUDE_DIRS})
target_link_libraries(webstreamer PUBLIC ${X264_LIBRARIES})
target_compile_definitions(webstreamer PUBLIC ${X264_DEFINITIONS})

# FFmpeg
find_package(FFmpeg COMPONENTS SWSCALE REQUIRED)
target_include_directories(webstreamer PUBLIC ${FFMPEG_INCLUDE_DIRS})
target_link_libraries(webstreamer PUBLIC ${FFMPEG_LIBRARIES})
target_compile_definitions(webstreamer PUBLIC ${FFMPEG_DEFINITIONS})

# # LibSourcey
# find_package(LibSourcey REQUIRED)
# target_include_directories(webstreamer PUBLIC ${LIBSOURCEY_INCLUDE_DIRS})
# target_link_libraries(webstreamer ${LIBSOURCEY_LIBRARIES})
# add_definitions(${LIBSOURCEY_DEFINITIONS})

# # WebRTC
find_package(WebRTC)
if (${WEBRTC_FOUND})
  target_include_directories(webstreamer PUBLIC ${WEBRTC_INCLUDE_DIRS})
  target_link_libraries(webstreamer ${WEBRTC_LIBRARIES})
  # add_definitions(${WEBRTC_DEFINITIONS})
  if (${WIN32})
    target_compile_definitions(webstreamer PUBLIC "-DWEBRTC_WIN")
  elseif (${UNIX})
    target_compile_definitions(webstreamer PUBLIC "-DWEBRTC_POSIX")
  endif ()
  target_compile_definitions(webstreamer PUBLIC "-DWEBSTREAMER_ENABLE_WEBRTC")
else()
  message(STATUS "WebRTC not found, building without WebRTC support.")
endif (${WEBRTC_FOUND})

configure_file(
  ${PROJECT_SOURCE_DIR}/cmake/webstreamer-config.cmake.in
  "${PROJECT_BINARY_DIR}/webstreamer-config.cmake" @ONLY)
export(TARGETS webstreamer
  FILE "${PROJECT_BINARY_DIR}/webstreamer-targets.cmake")

# set(CONF_INCLUDE_DIRS "${CURRENT_SOURCE_DIR}/include")
# configure_file(cmake/webstreamer-config.cmake.in
#   "${PROJECT_BINARY_DIR}/webstreamer-config.cmake" @ONLY)
# export(TARGETS webstreamer
#   FILE "${PROJECT_BINARY_DIR}/webstreamer-targets.cmake")

# --- tests --
add_subdirectory(tests)
