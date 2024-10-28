set( IS_CLANG "$<OR:$<CXX_COMPILER_ID:AppleClang>,$<CXX_COMPILER_ID:Clang>>" )
set( IS_GNU "$<CXX_COMPILER_ID:GNU>" )
set( IS_MSVC "$<CXX_COMPILER_ID:MSVC>" )

add_library( godot-cpp-test SHARED EXCLUDE_FROM_ALL )

target_sources( godot-cpp-test
        PRIVATE
        src/example.cpp
        src/example.h
        src/register_types.cpp
        src/register_types.h
        src/tests.h
)

target_link_libraries( godot-cpp-test
        PRIVATE
        godot-cpp::template_release )

target_compile_options( godot-cpp-test
        PRIVATE
        $<$<OR:${IS_CLANG},${IS_GNU}>:
            -Wwrite-strings
        >
)

target_link_options( godot-cpp-test
        PRIVATE
        $<$<OR:${IS_CLANG},${IS_GNU}>:
            $<$<NOT:$<BOOL:${APPLE}>>:-static-libgcc>
            -static-libstdc++
        >

        $<${IS_MSVC}: /WX > # treat link warnings as errors.
        $<${IS_MSVC}: /MANIFEST:NO > #We dont need a manifest
)

### Create the correct name (name.os.build_type.arch)
set( ARCH_KEYS "w64;x86_64;amd64" )
set( ARCH_VALUES "x86_64;x86_64;x86_64" )
set( SYSTEM_ARCH "$<LOWER_CASE:${CMAKE_SYSTEM_PROCESSOR}>")
set( ARCH_INDEX "$<LIST:FIND,${ARCH_KEYS},${SYSTEM_ARCH}>")
set( SYSTEM_ARCH "$<LIST:GET,${ARCH_VALUES},${ARCH_INDEX}>")

set( SYSTEM_NAME "$<LOWER_CASE:${CMAKE_SYSTEM_NAME}>$<$<BOOL:${ANDROID}>:.${ANDROID_ABI}>")

set_target_properties( godot-cpp-test
        PROPERTIES
        CXX_STANDARD 17
        CXX_EXTENSIONS OFF
        CXX_VISIBILITY_PRESET ${GODOT_SYMBOL_VISIBILITY}
        POSITION_INDEPENDENT_CODE ON
        BUILD_RPATH_USE_ORIGIN ON
        LINK_SEARCH_START_STATIC ON
        LINK_SEARCH_END_STATIC ON
        LIBRARY_OUTPUT_DIRECTORY "$<1:${CMAKE_CURRENT_SOURCE_DIR}/project/bin/>"
        RUNTIME_OUTPUT_DIRECTORY "$<1:${CMAKE_CURRENT_SOURCE_DIR}/project/bin/>"

        PREFIX lib
        OUTPUT_NAME "gdexample.${SYSTEM_NAME}.template_release.${SYSTEM_ARCH}"

        #MSVC Options
        PDB_OUTPUT_DIRECTORY "$<1:${CMAKE_CURRENT_SOURCE_DIR}/project/bin/>"
)
