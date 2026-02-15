cmake_minimum_required(VERSION 3.16)

# UTF-8 source
add_compile_options("$<$<AND:$<C_COMPILER_ID:MSVC>,$<COMPILE_LANGUAGE:C>>:/utf-8>")
add_compile_options("$<$<AND:$<CXX_COMPILER_ID:MSVC>,$<COMPILE_LANGUAGE:CXX>>:/utf-8>")
add_compile_options("$<$<AND:$<CXX_COMPILER_ID:MSVC>,$<COMPILE_LANGUAGE:CXX>>:/Zc:__cplusplus>")

# MSVC static runtime
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

set(PROJECT_NAME sigcheck)
set(PROJECT_VERSION 1.00)

project(${PROJECT_NAME} VERSION ${PROJECT_VERSION})

# vcpkgのlibtomcrypt/libtommathは.cmakeファイルを提供していないため
# find_package()がty変えないので自力でパスを構築する
find_path(LIBTOMCRYPT_INCLUDE_DIR tomcrypt.h)
find_path(LIBTOMMATH_INCLUDE_DIR tommath.h)

find_library(LIBTOMCRYPT_LIBRARY tomcrypt)
find_library(LIBTOMMATH_LIBRARY tommath)

if(TVP_STATIC_PLUGIN)
    set(TVP_LIBRARY_TYPE STATIC)
else()
    set(TVP_LIBRARY_TYPE SHARED)
endif()

add_library(${PROJECT_NAME} ${TVP_LIBRARY_TYPE}
    ${CMAKE_CURRENT_SOURCE_DIR}/../ncbind/ncbind.cpp
	main.cpp
)

if (TVP_STATIC_PLUGIN)
target_compile_definitions(${PROJECT_NAME} PRIVATE
    TVP_STATIC_PLUGIN
    TVP_PLUGIN_NAME=${PROJECT_NAME}
)
endif()

target_compile_definitions(${PROJECT_NAME} PRIVATE
	LTM_DESC
	USE_LTM

	# vcpkgのlibtommathでデッドコード除去がうまく働かずに
	# 未定義シンボル参照が起こる場合用のダミー定義を有効にする
	ENABLE_DUMMY_FOR_DEAD_CODE_ELIM_FAIL
)

target_include_directories(${PROJECT_NAME} PUBLIC
	${LIBTOMCRYPT_INCLUDE_DIR}
	${LIBTOMMATH_INCLUDE_DIR}
)

target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/../ncbind
)

target_link_libraries(${PROJECT_NAME} PUBLIC
    tp_stub
	${LIBTOMCRYPT_LIBRARY}
	${LIBTOMMATH_LIBRARY}
)
