cmake_minimum_required(VERSION 3.11.0 FATAL_ERROR)
project(libunwind)

include(FetchContent)
FetchContent_Declare(
  lzma
  GIT_REPOSITORY https://github.com/simpleton/lzma.git
  GIT_TAG        android_master
)
FetchContent_GetProperties(lzma)
if(NOT lzma_POPULATED)
  FetchContent_Populate(lzma)
  add_subdirectory(${lzma_SOURCE_DIR} ${lzma_BINARY_DIR})
endif()

if (NOT TARGET lzma)
  message(FATAL_ERROR, "Need lzma!!")
endif()

set(UNWIND_ROOT ${CMAKE_CURRENT_SOURCE_DIR})



set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-format")
add_definitions(-DHAVE_CONFIG_H -D_GNU_SOURCE)
if (${CMAKE_BUILD_TYPE} MATCHES debug)
else()
	add_definitions(-U_FORTIFY_SOURCE)
endif()

include_directories(${UNWIND_ROOT}/src ${UNWIND_ROOT}/include ${UNWIND_ROOT}/lzma/C)

#Common sources for all platforms
file(GLOB UNWIND_SOURCES_COMMON
    ${UNWIND_ROOT}/src/dl-iterate-phdr-android.c
    ${UNWIND_ROOT}/src/mi/init.c
    ${UNWIND_ROOT}/src/mi/flush_cache.c
    ${UNWIND_ROOT}/src/mi/mempool.c
    ${UNWIND_ROOT}/src/mi/strerror.c
    ${UNWIND_ROOT}/src/mi/backtrace.c
    ${UNWIND_ROOT}/src/mi/dyn-cancel.c
    ${UNWIND_ROOT}/src/mi/dyn-info-list.c
    ${UNWIND_ROOT}/src/mi/dyn-register.c
    ${UNWIND_ROOT}/src/mi/map.c
    ${UNWIND_ROOT}/src/mi/Lmap.c
    ${UNWIND_ROOT}/src/mi/Ldyn-extract.c
    ${UNWIND_ROOT}/src/mi/Lfind_dynamic_proc_info.c
    ${UNWIND_ROOT}/src/mi/Lget_proc_info_by_ip.c
    ${UNWIND_ROOT}/src/mi/Lget_proc_name.c
    ${UNWIND_ROOT}/src/mi/Lput_dynamic_unwind_info.c
    ${UNWIND_ROOT}/src/mi/Ldestroy_addr_space.c
    ${UNWIND_ROOT}/src/mi/Lget_reg.c
    ${UNWIND_ROOT}/src/mi/Lset_reg.c
    ${UNWIND_ROOT}/src/mi/Lget_fpreg.c
    ${UNWIND_ROOT}/src/mi/Lset_fpreg.c
    ${UNWIND_ROOT}/src/mi/Lset_caching_policy.c
    ${UNWIND_ROOT}/src/mi/Gdyn-extract.c
    ${UNWIND_ROOT}/src/mi/Gdyn-remote.c
    ${UNWIND_ROOT}/src/mi/Gfind_dynamic_proc_info.c
    ${UNWIND_ROOT}/src/mi/Gget_accessors.c
    ${UNWIND_ROOT}/src/mi/Gget_proc_info_by_ip.c
    ${UNWIND_ROOT}/src/mi/Gget_proc_name.c
    ${UNWIND_ROOT}/src/mi/Gput_dynamic_unwind_info.c
    ${UNWIND_ROOT}/src/mi/Gdestroy_addr_space.c
    ${UNWIND_ROOT}/src/mi/Gget_reg.c
    ${UNWIND_ROOT}/src/mi/Gset_reg.c
    ${UNWIND_ROOT}/src/mi/Gget_fpreg.c
    ${UNWIND_ROOT}/src/mi/Gset_fpreg.c
    ${UNWIND_ROOT}/src/mi/Gset_caching_policy.c
    ${UNWIND_ROOT}/src/dwarf/Lexpr.c
    ${UNWIND_ROOT}/src/dwarf/Lfde.c
    ${UNWIND_ROOT}/src/dwarf/Lparser.c
    ${UNWIND_ROOT}/src/dwarf/Lpe.c
    ${UNWIND_ROOT}/src/dwarf/Lstep_dwarf.c
    ${UNWIND_ROOT}/src/dwarf/Lfind_proc_info-lsb.c
    ${UNWIND_ROOT}/src/dwarf/Lfind_unwind_table.c
    ${UNWIND_ROOT}/src/dwarf/Gexpr.c
    ${UNWIND_ROOT}/src/dwarf/Gfde.c
    ${UNWIND_ROOT}/src/dwarf/Gfind_proc_info-lsb.c
    ${UNWIND_ROOT}/src/dwarf/Gfind_unwind_table.c
    ${UNWIND_ROOT}/src/dwarf/Gparser.c
    ${UNWIND_ROOT}/src/dwarf/Gpe.c
    ${UNWIND_ROOT}/src/dwarf/Gstep_dwarf.c
    ${UNWIND_ROOT}/src/dwarf/global.c
    ${UNWIND_ROOT}/src/os-common.c
    ${UNWIND_ROOT}/src/os-linux.c
    ${UNWIND_ROOT}/src/Los-common.c
    ${UNWIND_ROOT}/src/ptrace/_UPT_access_fpreg.c
    ${UNWIND_ROOT}/src/ptrace/_UPT_access_mem.c
    ${UNWIND_ROOT}/src/ptrace/_UPT_access_reg.c
    ${UNWIND_ROOT}/src/ptrace/_UPT_accessors.c
    ${UNWIND_ROOT}/src/ptrace/_UPT_create.c
    ${UNWIND_ROOT}/src/ptrace/_UPT_destroy.c
    ${UNWIND_ROOT}/src/ptrace/_UPT_elf.c
    ${UNWIND_ROOT}/src/ptrace/_UPT_find_proc_info.c
    ${UNWIND_ROOT}/src/ptrace/_UPT_get_dyn_info_list_addr.c
    ${UNWIND_ROOT}/src/ptrace/_UPT_get_proc_name.c
    ${UNWIND_ROOT}/src/ptrace/_UPT_put_unwind_info.c
    ${UNWIND_ROOT}/src/ptrace/_UPT_reg_offset.c
    ${UNWIND_ROOT}/src/ptrace/_UPT_resume.c
)

#Platform-specific sources
if (${ANDROID_ABI} MATCHES x86)
    include_directories(${UNWIND_ROOT}/include/tdep-x86)
    file(GLOB UNWIND_SOURCES_ARCH
        ${UNWIND_ROOT}/src/x86/Gcreate_addr_space.c
        ${UNWIND_ROOT}/src/x86/Gget_proc_info.c
        ${UNWIND_ROOT}/src/x86/Gget_save_loc.c
        ${UNWIND_ROOT}/src/x86/Gglobal.c
        ${UNWIND_ROOT}/src/x86/Ginit.c
        ${UNWIND_ROOT}/src/x86/Ginit_local.c
        ${UNWIND_ROOT}/src/x86/Ginit_remote.c
        ${UNWIND_ROOT}/src/x86/Gos-linux.c
        ${UNWIND_ROOT}/src/x86/Gregs.c
        ${UNWIND_ROOT}/src/x86/Gresume.c
        ${UNWIND_ROOT}/src/x86/Gstep.c
        ${UNWIND_ROOT}/src/x86/Lcreate_addr_space.c
        ${UNWIND_ROOT}/src/x86/Lget_proc_info.c
        ${UNWIND_ROOT}/src/x86/Lget_save_loc.c
        ${UNWIND_ROOT}/src/x86/Lglobal.c
        ${UNWIND_ROOT}/src/x86/Linit.c
        ${UNWIND_ROOT}/src/x86/Linit_local.c
        ${UNWIND_ROOT}/src/x86/Linit_remote.c
        ${UNWIND_ROOT}/src/x86/Los-linux.c
        ${UNWIND_ROOT}/src/x86/Lregs.c
        ${UNWIND_ROOT}/src/x86/Lresume.c
        ${UNWIND_ROOT}/src/x86/Lstep.c
        ${UNWIND_ROOT}/src/x86/is_fpreg.c
        ${UNWIND_ROOT}/src/x86/regname.c
        ${UNWIND_ROOT}/src/elf32.c
     )
    file(GLOB UNWIND_SOURCES_ASM
        ${UNWIND_ROOT}/src/x86/getcontext-linux.S
    )
elseif(${CMAKE_SYSTEM_PROCESSOR} MATCHES x86_64)
    include_directories(${UNWIND_ROOT}/include/tdep-x86_64)
    file(GLOB UNWIND_SOURCES_ARCH
        ${UNWIND_ROOT}/src/x86_64/Gcreate_addr_space.c
        ${UNWIND_ROOT}/src/x86_64/Gget_proc_info.c
        ${UNWIND_ROOT}/src/x86_64/Gget_save_loc.c
        ${UNWIND_ROOT}/src/x86_64/Gglobal.c
        ${UNWIND_ROOT}/src/x86_64/Ginit.c
        ${UNWIND_ROOT}/src/x86_64/Ginit_local.c
        ${UNWIND_ROOT}/src/x86_64/Ginit_remote.c
        ${UNWIND_ROOT}/src/x86_64/Gos-linux.c
        ${UNWIND_ROOT}/src/x86_64/Gregs.c
        ${UNWIND_ROOT}/src/x86_64/Gresume.c
        ${UNWIND_ROOT}/src/x86_64/Gstash_frame.c
        ${UNWIND_ROOT}/src/x86_64/Gstep.c
        ${UNWIND_ROOT}/src/x86_64/Gtrace.c
        ${UNWIND_ROOT}/src/x86_64/Lcreate_addr_space.c
        ${UNWIND_ROOT}/src/x86_64/Lget_proc_info.c
        ${UNWIND_ROOT}/src/x86_64/Lget_save_loc.c
        ${UNWIND_ROOT}/src/x86_64/Lglobal.c
        ${UNWIND_ROOT}/src/x86_64/Linit.c
        ${UNWIND_ROOT}/src/x86_64/Linit_local.c
        ${UNWIND_ROOT}/src/x86_64/Linit_remote.c
        ${UNWIND_ROOT}/src/x86_64/Los-linux.c
        ${UNWIND_ROOT}/src/x86_64/Lregs.c
        ${UNWIND_ROOT}/src/x86_64/Lresume.c
        ${UNWIND_ROOT}/src/x86_64/Lstash_frame.c
        ${UNWIND_ROOT}/src/x86_64/Lstep.c
        ${UNWIND_ROOT}/src/x86_64/Ltrace.c
        ${UNWIND_ROOT}/src/x86_64/is_fpreg.c
        ${UNWIND_ROOT}/src/x86_64/regname.c
        ${UNWIND_ROOT}/src/elf64.c
    )
    file(GLOB UNWIND_SOURCES_ASM
        ${UNWIND_ROOT}/src/x86_64/getcontext.S
        ${UNWIND_ROOT}/src/x86_64/setcontext.S
    )
elseif(${ANDROID_ABI} MATCHES arm)
    include_directories(${UNWIND_ROOT}/include/tdep-arm)
    file(GLOB UNWIND_SOURCES_ARCH
        ${UNWIND_ROOT}/src/arm/Gapply_reg_state.c
        ${UNWIND_ROOT}/src/arm/Gcreate_addr_space.c
        ${UNWIND_ROOT}/src/arm/gen-offsets.c
        ${UNWIND_ROOT}/src/arm/getcontext.S
        ${UNWIND_ROOT}/src/arm/Gex_tables.c
        ${UNWIND_ROOT}/src/arm/Gget_proc_info.c
        ${UNWIND_ROOT}/src/arm/Gget_save_loc.c
        ${UNWIND_ROOT}/src/arm/Gglobal.c
        ${UNWIND_ROOT}/src/arm/Ginit.c
        ${UNWIND_ROOT}/src/arm/Ginit_local.c
        ${UNWIND_ROOT}/src/arm/Ginit_remote.c
        ${UNWIND_ROOT}/src/arm/Gos-linux.c
        ${UNWIND_ROOT}/src/arm/Gregs.c
        ${UNWIND_ROOT}/src/arm/Greg_states_iterate.c
        ${UNWIND_ROOT}/src/arm/Gresume.c
        ${UNWIND_ROOT}/src/arm/Gstash_frame.c
        ${UNWIND_ROOT}/src/arm/Gstep.c
        ${UNWIND_ROOT}/src/arm/Gtrace.c
        ${UNWIND_ROOT}/src/arm/init.h
        ${UNWIND_ROOT}/src/arm/is_fpreg.c
        ${UNWIND_ROOT}/src/arm/offsets.h
        ${UNWIND_ROOT}/src/arm/regname.c
        ${UNWIND_ROOT}/src/arm/siglongjmp.S
        ${UNWIND_ROOT}/src/arm/unwind_i.h
        ${UNWIND_ROOT}/src/elf32.c
    )
    file(GLOB UNWIND_SOURCES_ASM
        ${UNWIND_ROOT}/src/arm/getcontext.S
    )
elseif(${ANDROID_ABI} MATCHES aarch64)
    include_directories(${UNWIND_ROOT}/include/tdep-aarch64)
    file(GLOB UNWIND_SOURCES_ARCH
        ${UNWIND_ROOT}/src/aarch64/Gapply_reg_state.c
        ${UNWIND_ROOT}/src/aarch64/Gcreate_addr_space.c
        ${UNWIND_ROOT}/src/aarch64/gen-offsets.c
        ${UNWIND_ROOT}/src/aarch64/getcontext.S
        ${UNWIND_ROOT}/src/aarch64/Gget_proc_info.c
        ${UNWIND_ROOT}/src/aarch64/Gget_save_loc.c
        ${UNWIND_ROOT}/src/aarch64/Gglobal.c
        ${UNWIND_ROOT}/src/aarch64/Ginit.c
        ${UNWIND_ROOT}/src/aarch64/Ginit_local.c
        ${UNWIND_ROOT}/src/aarch64/Ginit_remote.c
        ${UNWIND_ROOT}/src/aarch64/Gis_signal_frame.c
        ${UNWIND_ROOT}/src/aarch64/Gregs.c
        ${UNWIND_ROOT}/src/aarch64/Greg_states_iterate.c
        ${UNWIND_ROOT}/src/aarch64/Gresume.c
        ${UNWIND_ROOT}/src/aarch64/Gstash_frame.c
        ${UNWIND_ROOT}/src/aarch64/Gstep.c
        ${UNWIND_ROOT}/src/aarch64/Gtrace.c
        ${UNWIND_ROOT}/src/aarch64/init.h
        ${UNWIND_ROOT}/src/aarch64/is_fpreg.c
        ${UNWIND_ROOT}/src/aarch64/offsets.h
        ${UNWIND_ROOT}/src/aarch64/regname.c
        ${UNWIND_ROOT}/src/aarch64/siglongjmp.S
        ${UNWIND_ROOT}/src/aarch64/unwind_i.h
        ${UNWIND_ROOT}/src/elf64.c
    )
else()
    message(FATAL_ERROR "Unsupported architecture: ${ANDROID_ABI}")
endif()

set_source_files_properties(${UNWIND_SOURCES_ASM} PROPERTIES LANGUAGE C)
set(OUTPUT_DIR ${CMAKE_BINARY_DIR}/../dist/${ANDROID_ABI}/${REL_OUTPUT_DIR})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${OUTPUT_DIR}")

set(REL_OUTPUT_DIR "${CMAKE_BUILD_TYPE}/${ANDROID_ABI}")

add_library(libunwind STATIC ${UNWIND_SOURCES_COMMON} ${UNWIND_SOURCES_ARCH} ${UNWIND_SOURCES_ASM})

target_link_libraries(libunwind lzma)
