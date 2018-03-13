cmake_minimum_required(VERSION 3.1)
project(codejam_bootstrap LANGUAGES CXX)

set(MY_MOTTO "OMAE_WA_MOU_SHINDEIRU" CACHE STRING "your personal motto (no whitespace!)")
set(PROBLEMS "" CACHE STRING ";-separated problems list")
set(HIGHTAIL_VERSION "v0.9.6" CACHE STRING "hightail version")
set(HIGHTAIL_DIST "https://github.com/dj3500/hightail/releases/download/${HIGHTAIL_VERSION}/Hightail-${HIGHTAIL_VERSION}.jar")

if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/hightail.jar")
  file(DOWNLOAD ${HIGHTAIL_DIST} ${CMAKE_CURRENT_SOURCE_DIR}/hightail.jar)
endif()

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/hightail.config.in ${CMAKE_CURRENT_SOURCE_DIR}/hightail.config @ONLY) 

foreach(X ${PROBLEMS})
  set(X_DIR ${CMAKE_CURRENT_SOURCE_DIR}/${X})
  add_executable(${X} ${X_DIR}/main.cpp)
  target_compile_features(${X} PRIVATE cxx_std_17)
  target_compile_definitions(${X} PRIVATE "$<$<CONFIG:Debug>:${MY_MOTTO}>")
  add_custom_command(
    TARGET ${X} 
    POST_BUILD 
    COMMAND ${CMAKE_COMMAND} -E create_symlink $<TARGET_FILE:${X}> ${X_DIR}/$<IF:$<CONFIG:Debug>,"deb","rel">${CMAKE_EXECUTABLE_SUFFIX}
  )
  if(NOT EXISTS ${X_DIR})
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/main.cpp.in ${X_DIR}/main.cpp @ONLY) 
    file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/check.py DESTINATION ${X_DIR})
    file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/sample.txt DESTINATION ${X_DIR})
  endif()
endforeach(X)
