set(TPSTUB_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../tp_stub CACHE PATH "Path to tp_stub directory")
include(${TPSTUB_DIR}/krkrz.cmake)

# vcpkgのlibtomcrypt/libtommathは.cmakeファイルを提供していないため
# find_package()が使えないので自力でパスを構築する
find_path(LIBTOMCRYPT_INCLUDE_DIR tomcrypt.h)
find_path(LIBTOMMATH_INCLUDE_DIR tommath.h)

find_library(LIBTOMCRYPT_LIBRARY tomcrypt)
find_library(LIBTOMMATH_LIBRARY tommath)

krkrz_plugin(sigcheck
	NCBIND
	SOURCES
		main.cpp
	INCLUDES
		${LIBTOMCRYPT_INCLUDE_DIR}
		${LIBTOMMATH_INCLUDE_DIR}
	LIBRARIES
		${LIBTOMCRYPT_LIBRARY}
		${LIBTOMMATH_LIBRARY}
)

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

target_link_libraries(${PROJECT_NAME} PUBLIC
	${LIBTOMCRYPT_LIBRARY}
	${LIBTOMMATH_LIBRARY}
)
