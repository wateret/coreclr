project(linuxnonjit)

add_definitions(-DALT_JIT)
add_definitions(-DFEATURE_NO_HOST)
add_definitions(-DSELF_NO_HOST)
remove_definitions(-DFEATURE_MERGE_JIT_AND_ENGINE)

if(FEATURE_READYTORUN)
  add_definitions(-DFEATURE_READYTORUN_COMPILER)
endif(FEATURE_READYTORUN)

if (CLR_CMAKE_PLATFORM_ARCH_I386)
    remove_definitions(-DFEATURE_SIMD)
    add_definitions(-DUNIX_X86_ABI)
    set(JIT_ARCH_ALTJIT_SOURCES ${JIT_I386_SOURCES})
elseif(CLR_CMAKE_PLATFORM_ARCH_AMD64)
    add_definitions(-DUNIX_AMD64_ABI)
    add_definitions(-DFEATURE_UNIX_AMD64_STRUCT_PASSING)
    set(JIT_ARCH_ALTJIT_SOURCES ${JIT_AMD64_SOURCES})
else()
    clr_unknown_arch()
endif()

if(WIN32)
  add_definitions(-DFX_VER_INTERNALNAME_STR=linuxnonjit.dll)
endif(WIN32)

add_library_clr(linuxnonjit
   SHARED
   ${SHARED_LIB_SOURCES}
   ${JIT_ARCH_ALTJIT_SOURCES}
)

add_dependencies(linuxnonjit jit_exports)

set_property(TARGET linuxnonjit APPEND_STRING PROPERTY LINK_FLAGS ${JIT_EXPORTS_LINKER_OPTION})
set_property(TARGET linuxnonjit APPEND_STRING PROPERTY LINK_DEPENDS ${JIT_EXPORTS_FILE})

set(RYUJIT_LINK_LIBRARIES
   utilcodestaticnohost
   gcinfo
)

if(CLR_CMAKE_PLATFORM_UNIX)
    list(APPEND RYUJIT_LINK_LIBRARIES
       mscorrc_debug
       coreclrpal
       palrt
    )
else()
    list(APPEND RYUJIT_LINK_LIBRARIES
       ${STATIC_MT_CRT_LIB}
       ${STATIC_MT_VCRT_LIB}
       kernel32.lib
       advapi32.lib
       ole32.lib
       oleaut32.lib
       uuid.lib
       user32.lib
       version.lib
       shlwapi.lib
       bcrypt.lib
       crypt32.lib
       RuntimeObject.lib
    )
endif(CLR_CMAKE_PLATFORM_UNIX)

target_link_libraries(linuxnonjit
   ${RYUJIT_LINK_LIBRARIES}
)

# add the install targets
install_clr(linuxnonjit)
