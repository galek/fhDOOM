cmake_minimum_required (VERSION 3.2.0)
project (fhDOOM)

OPTION(ID_ALLOW_TOOLS "Compile Tools" ON)
OPTION(ID_ALLOW_MAYA "Compile Maya Tools" ON)
OPTION(ID_UNICODE "Use unicode version of WIN32 API" OFF)

MACRO(add_msvc_precompiled_header PrecompiledHeader SourcesVar)
  IF(MSVC)
    GET_FILENAME_COMPONENT(PrecompiledBasename ${PrecompiledHeader} NAME_WE)

    SET(PrecompiledSource ${CMAKE_CURRENT_BINARY_DIR}/precompiled.cpp)    
    SET(PrecompiledBinary ${CMAKE_CURRENT_BINARY_DIR}/precompiled.pch)      

    FILE(WRITE ${PrecompiledSource} "//generated cpp for precompiled header, DO NOT EDIT!\n#include \"${PrecompiledHeader}\"" )

    SET(Sources ${${SourcesVar}})

    set_property(
      SOURCE ${PrecompiledSource}
      APPEND_STRING 
      PROPERTY COMPILE_FLAGS " /Yc\"${PrecompiledHeader}\" /Fp\"${PrecompiledBinary}\" "
    )

    set_property(
      SOURCE ${Sources}
      APPEND_STRING 
      PROPERTY COMPILE_FLAGS " /Yu\"${PrecompiledHeader}\" /Fp\"${PrecompiledBinary}\" "
    )    

    # Add precompiled header to SourcesVar
    LIST(APPEND ${SourcesVar} ${PrecompiledSource})
  ENDIF(MSVC)
ENDMACRO()

MACRO(add_force_include Header SourcesVar)
  SET(Sources ${${SourcesVar}})
  IF(MSVC)
    set_property(
      SOURCE ${Sources}
      APPEND_STRING 
      PROPERTY COMPILE_FLAGS " /FI\"${Header}\" "
    )
  ELSE()
    set_property(
      SOURCE ${Sources}
      APPEND_STRING 
      PROPERTY COMPILE_FLAGS " -include \"${Header}\" "
    )    
  ENDIF()
ENDMACRO()

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

IF(MSVC)
  IF(ID_ALLOW_TOOLS)
    add_definitions(-DID_ALLOW_TOOLS)
  ENDIF()

  IF(ID_UNICODE)
    add_definitions(-D_UNICODE)
  ENDIF()

  SET(CMAKE_EXE_LINKER_FLAGS "/STACK:16777216,16777216")

  SET(CMAKE_CXX_FLAGS         "/DWIN32 /D_WIN32 /D_WINDOWS /EHsc /GR /fp:fast /GS- /MP /W3 /D_USE_32BIT_TIME_T /D_CRT_SECURE_NO_DEPRECATE /D_CRT_NONSTDC_NO_DEPRECATE")    
  SET(CMAKE_CXX_FLAGS_DEBUG   "/D_DEBUG /MTd /Od /Zi")
  SET(CMAKE_CXX_FLAGS_RELEASE "/MT /O2 /GL")  
  SET(CMAKE_CXX_FLAGS_RELWITHDEBINFO "/MT /O2 /GL /Zi")

  SET(CMAKE_C_FLAGS         ${CMAKE_CXX_FLAGS})    
  SET(CMAKE_C_FLAGS_DEBUG   ${CMAKE_CXX_FLAGS_DEBUG})
  SET(CMAKE_C_FLAGS_RELEASE ${CMAKE_CXX_FLAGS_RELEASE})    
  SET(CMAKE_C_FLAGS_RELWITHDEBINFO ${CMAKE_CXX_FLAGS_RELWITHDEBINFO})        

  SET_PROPERTY(GLOBAL PROPERTY PREDEFINED_TARGETS_FOLDER  "")
  SET_PROPERTY(GLOBAL PROPERTY USE_FOLDERS ON)
ENDIF()  

IF(CMAKE_COMPILER_IS_GNUCC)
  add_definitions(-DID_ENABLE_CURL=0)
  add_definitions(-DXTHREADS)
  SET(CMAKE_CXX_FLAGS "-std=c++11 -Wno-format-security -Wno-invalid-offsetof -fvisibility=hidden -fno-unsafe-math-optimizations -fno-strict-aliasing -fPIC")
  SET(CMAKE_CXX_FLAGS_DEBUG "-g -fno-omit-frame-pointer -O0")
  SET(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG -msse2")
ENDIF()

IF(UNIX)
    SET(CMAKE_SHARED_LIBRARY_PREFIX "")
ENDIF()

include_directories(neo/renderer)
add_subdirectory (neo/idlib)
IF(NOT CMAKE_COMPILER_IS_GNUCC)
  add_subdirectory (neo/curl)
  add_subdirectory (neo/TypeInfo)
  if(ID_ALLOW_MAYA)
    add_subdirectory (neo/maya2md5)
  endif()
ENDIF()  
add_subdirectory (neo/game)
add_subdirectory (neo/d3xp)
add_subdirectory (neo)


