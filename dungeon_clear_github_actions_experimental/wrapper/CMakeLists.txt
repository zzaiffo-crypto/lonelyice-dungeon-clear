# Experimental wrapper for LonelyIce. NOT tested against Windows binary.
include(FetchContent)
find_package(Git REQUIRED)
file(READ "${CMAKE_CURRENT_SOURCE_DIR}/plugin.json" manifest)
string(JSON plugin_id GET "${manifest}" id)
string(JSON repo GET "${manifest}" source repo)
string(JSON revision GET "${manifest}" source commit)
if (revision MATCHES "^0+$")
  message(FATAL_ERROR "Pin upstream commit in plugin.json first (the Actions workflow does this).")
endif()

file(GLOB patches "${CMAKE_CURRENT_SOURCE_DIR}/patches/*.patch")
set(patch_command "")
if (patches)
  set(patch_command PATCH_COMMAND "${GIT_EXECUTABLE}" reset --quiet --hard
     COMMAND "${GIT_EXECUTABLE}" clean --quiet -fd
     COMMAND "${GIT_EXECUTABLE}" apply --whitespace=nowarn ${patches})
endif()
FetchContent_Declare(dungeon_clear_upstream
  GIT_REPOSITORY "${repo}"
  GIT_TAG "${revision}"
  SOURCE_SUBDIR __no_cmake__
  ${patch_command}
)
FetchContent_MakeAvailable(dungeon_clear_upstream)
set(module "${dungeon_clear_upstream_SOURCE_DIR}")
file(GLOB_RECURSE dc_sources CONFIGURE_DEPENDS "${module}/src/*.cpp")
if (NOT dc_sources)
  message(FATAL_ERROR "No mod-dungeon-clear sources found: ${module}")
endif()

# Playerbots is a separate shared library in LonelyIce.
AddPlugin(mod-dungeon-clear SOURCES ${dc_sources} plugin/plugin.cpp LINK mod-playerbots)
if (NOT TARGET mod-dungeon-clear)
  return()
endif()
target_include_directories(mod-dungeon-clear PRIVATE
  "${module}/src"
  "${module}/src/Ai/Dungeon/DungeonClear"
  "${module}/src/Ai/Dungeon/DungeonClear/Util"
)
# Dungeon Clear upstream uses this workaround when compiling with MSVC.
if (MSVC)
  target_compile_options(mod-dungeon-clear PRIVATE /D_USE_MATH_DEFINES=)
endif()

# Detect API variants in the core (the original module uses matching probes).
if (DEFINED LONELYICE_CORE_DIR)
  set(dc_core "${LONELYICE_CORE_DIR}")
else()
  set(dc_core "${CMAKE_SOURCE_DIR}")
endif()
set(dc_compat "")
set(dc_server_header "${dc_core}/src/server/game/Scripting/ScriptDefines/ServerScript.h")
if (EXISTS "${dc_server_header}")
  file(READ "${dc_server_header}" contents)
  string(FIND "${contents}" "SERVERHOOK_ON_PACKET_SENT" idx)
  if (NOT idx EQUAL -1)
    list(APPEND dc_compat DC_CORE_HAS_ON_PACKET_SENT=1)
  endif()
endif()
set(dc_world_header "${dc_core}/src/server/game/World/IWorld.h")
if (EXISTS "${dc_world_header}")
  file(READ "${dc_world_header}" contents)
  string(FIND "${contents}" "AddQueryHolderCallback" idx)
  if (NOT idx EQUAL -1)
    list(APPEND dc_compat DC_CORE_IWORLD_HAS_QUERYHOLDER=1)
  endif()
endif()
if (dc_compat)
  target_compile_definitions(mod-dungeon-clear PRIVATE ${dc_compat})
endif()

# Keep the module's config with the compiled plugin.
set(copy_commands "")
foreach(folder conf data)
  if (EXISTS "${module}/${folder}")
    list(APPEND copy_commands
      COMMAND ${CMAKE_COMMAND} -E copy_directory
        "${module}/${folder}" "${AC_PLUGINS_OUTPUT_DIR}/${plugin_id}/${folder}")
  endif()
endforeach()
if (copy_commands)
  add_custom_target(dungeon-clear-module-files ALL ${copy_commands} VERBATIM)
  set_target_properties(dungeon-clear-module-files PROPERTIES FOLDER "plugins")
  add_dependencies(dungeon-clear-module-files mod-dungeon-clear-files)
  add_dependencies(mod-dungeon-clear dungeon-clear-module-files)
endif()
