
cmake_minimum_required (VERSION 2.8.11)

if (NOT DEFINED CMAKE_BUILD_TYPE)
  set (CMAKE_BUILD_TYPE Release CACHE STRING "Build type")
endif ()

project (OptPP)

#==============================================================================
# version
set (OPTPP_MAJOR_VERSION "2")
set (OPTPP_MINOR_VERSION "5")
set (OPTPP_BUGFIX_VERSION "0")
set (OPTPP_VERSION_STRING ${OPTPP_MAJOR_VERSION}.${OPTPP_MINOR_VERSION}.${OPTPP_BUGFIX_VERSION})
message (STATUS "OptPP version ${OPTPP_VERSION_STRING}")

list (APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

option (BUILD_SHARED_LIBS "Build OptPP as a shared library" OFF)
option (OPTPP_TESTS "build unit tests" OFF)
option (BUILD_32 "build for 32 bit arch" OFF)

include (GNUInstallDirs)



if(BUILD_32)
  add_compile_options(-m32)
  set (CMAKE_INSTALL_LIBDIR "/usr/local/lib/")
endif()

set (CMAKE_INSTALL_INCLUDEDIR "/usr/local/include/optpp_newmat")

# Offer the user the choice of overriding the installation directories
set (INSTALL_LIB_DIR     ${CMAKE_INSTALL_LIBDIR}        CACHE PATH "Installation directory for libraries")
set (INSTALL_BIN_DIR     ${CMAKE_INSTALL_BINDIR}        CACHE PATH "Installation directory for executables")
set (INSTALL_INCLUDE_DIR ${CMAKE_INSTALL_INCLUDEDIR}    CACHE PATH "Installation directory for header files")
set (INSTALL_DATA_DIR    ${CMAKE_INSTALL_DATADIR}/nlopt CACHE PATH "Installation directory for data files")
set (INSTALL_MAN_DIR     ${CMAKE_INSTALL_MANDIR}        CACHE PATH "Installation directory for man documentation")
set (INSTALL_CMAKE_DIR   ${INSTALL_LIB_DIR}/cmake/nlopt CACHE PATH "Installation directory for cmake config files")

# Make relative paths absolute (needed later on)
foreach (p LIB BIN INCLUDE DATA CMAKE)
  set (var INSTALL_${p}_DIR)
  set (RELATIVE_INSTALL_${p}_DIR ${INSTALL_${p}_DIR})
  if (NOT IS_ABSOLUTE "${${var}}")
    set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
  endif ()
endforeach ()


set (CMAKE_INSTALL_RPATH ${INSTALL_LIB_DIR})
set (CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
if(POLICY CMP0042)
  # Set MACOSX_RPATH to ON
  cmake_policy(SET CMP0042 NEW)
endif()


include (CheckIncludeFile)
include (CheckFunctionExists)
# include (CheckTypeSize)
# include (CheckCCompilerFlag)
# include (CheckCXXSymbolExists)
# include (CheckCXXCompilerFlag)
include (CheckLibraryExists)
# 
#==============================================================================
# COMPILATION CHECKINGS and CONFIGURATION GENERATION
#==============================================================================
set (HAVE_NAMESPACES 1)
set (HAVE_STD 1)
# check_include_file (dlfcn.h HAVE_DLFCN_H)
# check_include_file (getopt.h HAVE_GETOPT_H)
# check_include_file (unistd.h HAVE_UNISTD_H)
# check_include_file (string.h HAVE_STRING_H)
# check_include_file (strings.h HAVE_STRINGS_H)
# check_include_file (inttypes.h HAVE_INTTYPES_H)
# check_include_file (memory.h HAVE_MEMORY_H)
# check_include_file (stdlib.h HAVE_STDLIB_H)
# check_include_file (stdint.h HAVE_STDINT_H)
# check_include_file (time.h HAVE_TIME_H)
# check_include_file (sys/types.h HAVE_SYS_TYPES_H)
# check_include_file (sys/stat.h HAVE_SYS_STAT_H)
check_include_file (sys/time.h HAVE_SYS_TIME_H)
check_include_file (sys/times.h HAVE_SYS_TIMES_H)
check_include_file (sys/param.h HAVE_SYS_PARAM_H)
check_include_file (sys/resource.h HAVE_SYS_RESOURCE_H)

# if (HAVE_TIME_H AND HAVE_SYS_TIME_H)
#   set (TIME_WITH_SYS_TIME TRUE)
# endif ()
# check_function_exists (getpid HAVE_GETPID)
# check_function_exists (syscall HAVE_GETTID_SYSCALL)
# check_function_exists (isinf HAVE_ISINF)
# check_function_exists (isnan HAVE_ISNAN)
check_function_exists (gettimeofday HAVE_GETTIMEOFDAY)
# check_function_exists (qsort_r HAVE_QSORT_R)
# check_function_exists (time HAVE_TIME)
check_function_exists (times HAVE_TIMES)
# check_function_exists (copysign HAVE_COPYSIGN)
# check_type_size ("uint32_t" SIZEOF_UINT32_T)
# set (HAVE_UINT32_T ${SIZEOF_UINT32_T})
# check_type_size ("unsigned int" SIZEOF_UNSIGNED_INT)
# check_type_size ("unsigned long" SIZEOF_UNSIGNED_LONG)
# 
check_library_exists ("m" sqrt "" HAVE_LIBM)
if (HAVE_LIBM)
  set (M_LIBRARY m)
endif()

find_package (BLAS)
if (BLAS_FOUND)
  set (HAVE_BLAS 1)
endif ()

#==============================================================================
# CREATE nlopt_config.h
#==============================================================================

configure_file (include/OPT++_config.h.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/include/OPT++_config.h)

# pkgconfig file
# if (UNIX OR MINGW)
#   configure_file (${CMAKE_CURRENT_SOURCE_DIR}/nlopt.pc.in ${CMAKE_CURRENT_BINARY_DIR}/nlopt.pc @ONLY)
#   install (FILES ${CMAKE_CURRENT_BINARY_DIR}/nlopt.pc DESTINATION ${RELATIVE_INSTALL_LIB_DIR}/pkgconfig)
# endif ()

#==============================================================================
# nlopt LIBRARY TARGET (SHARED OR STATIC)
#==============================================================================

file (GLOB_RECURSE OPTPP_HEADERS "*.h" include)

set (OPTPP_SOURCES
src/globals.C
src/Base/Appl_Data.C
src/Base/Appl_Data_NPSOL.C
src/Base/FDNLF1.C
src/Base/LSQNLF.C
src/Base/NLF0.C
src/Base/NLF1.C
src/Base/NLF2.C
src/Base/NLP.C
src/Base/NLP0.C
src/Base/NLP1.C
src/Base/NLP2.C
src/Base/TOLS.C
src/Base/backtrack.C
src/Base/dogleg.C
src/Base/linesearch.C
src/Base/mcsrch.C
src/Base/trustpds.C
src/Base/trustregion.C
src/Constraints/BoundConstraint.C
src/Constraints/CompoundConstraint.C
src/Constraints/Constraint.C
src/Constraints/LinearConstraint.C
src/Constraints/LinearEquation.C
src/Constraints/LinearInequality.C
src/Constraints/NonLinearConstraint.C
src/Constraints/NonLinearEquation.C
src/Constraints/NonLinearInequality.C
src/GSS/GenSetBase.C
src/GSS/GenSetBox2d.C
src/GSS/GenSetMin.C
src/GSS/GenSetStd.C
src/GSS/OptGSS.C
src/Newton/OptBCEllipsoid.C
src/Newton/OptBCFDNewton.C
src/Newton/OptBCNewton.C
src/Newton/OptBCNewtonLike.C
src/Newton/OptBCQNewton.C
src/Newton/OptBaNewton.C
src/Newton/OptBaQNewton.C
src/Newton/OptCG.C
src/Newton/OptConstrFDNewton.C
src/Newton/OptConstrNewton.C
src/Newton/OptConstrNewtonLike.C
src/Newton/OptConstrQNewton.C
src/Newton/OptDHNIPS.C
src/Newton/OptFDNIPS.C
src/Newton/OptFDNewton.C
src/Newton/OptLBFGS.C
src/Newton/OptNIPS.C
src/Newton/OptNIPSLike.C
# src/Newton/OptNPSOL.C
src/Newton/OptNewton.C
src/Newton/OptNewtonLike.C
src/Newton/OptQNIPS.C
src/Newton/OptQNewton.C
# src/Newton/npsol_setup.c
src/PDS/create_scheme.C
src/PDS/pds.C
src/PDS/pdschk.C
src/PDS/pdsinit.C
src/PDS/pdsopt.C
src/PDS/pdsstep.C
src/PDS/pdswork.C
src/PDS/dqrdc.c
src/PDS/make_search.c
src/PDS/pdscld.c
src/PDS/pdscom.c
src/PDS/pdsdgn.c
src/PDS/pdsdone.c
src/PDS/pdseql.c
src/PDS/pdsget.c
# src/PDS/pdsglb.c
# src/PDS/pdsgop.c
src/PDS/pdshrk.c
src/PDS/pdslen.c
src/PDS/pdsrgt.c
src/PDS/pdsupd.c
# src/PDS/pdswap.c
src/PDS/sort.c
src/PDS/writes.c
src/Utils/BoolVector.C
src/Utils/OptppExceptions.C
src/Utils/OptppFatalError.C
src/Utils/ioformat.C
src/Utils/mcholesky.C
src/Utils/print.C
src/Utils/file_cutils.c
src/Utils/linalg.c
src/Utils/timers.c
newmat11/bandmat.C
newmat11/cholesky.C
newmat11/evalue.C
newmat11/fft.C
newmat11/hholder.C
newmat11/jacobi.C
newmat11/myexcept.C
newmat11/newfft.C
newmat11/newmat1.C
newmat11/newmat2.C
newmat11/newmat3.C
newmat11/newmat4.C
newmat11/newmat5.C
newmat11/newmat6.C
newmat11/newmat7.C
newmat11/newmat8.C
newmat11/newmat9.C
newmat11/newmatex.C
newmat11/newmatnl.C
newmat11/newmatrm.C
newmat11/solution.C
newmat11/sort.C
newmat11/submat.C
newmat11/svd.C)

install (FILES ${OPTPP_HEADERS} DESTINATION ${RELATIVE_INSTALL_INCLUDE_DIR})

add_library (opt ${OPTPP_SOURCES})
# target_link_libraries (opt ${M_LIBRARY})

# set_target_properties (opt PROPERTIES SOVERSION ${SO_MAJOR})
# set_target_properties (opt PROPERTIES VERSION "${SO_MAJOR}.${SO_MINOR}.${SO_PATCH}")

target_compile_definitions (opt PUBLIC HAVE_CONFIG_H)


#==============================================================================
# INCLUDE DIRECTORIES
#==============================================================================
target_include_directories (opt PRIVATE
  include
  newmat11
  ${PROJECT_BINARY_DIR}/src/api
  ${PROJECT_BINARY_DIR}/include)
# 
# get_target_property (OPTPP_PRIVATE_INCLUDE_DIRS ${nlopt_lib} INCLUDE_DIRECTORIES)
# target_include_directories (${nlopt_lib} INTERFACE "$<BUILD_INTERFACE:${PROJECT_BINARY_DIR}/src/api>" "$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>")

if (BUILD_SHARED_LIBS)
  target_compile_definitions (opt PUBLIC -DOPTPP_DLL)
  target_compile_definitions (opt PRIVATE -DOPTPP_DLL_EXPORT)
endif ()

# # pass -fPIC in case swig module is built with static library
# if (NOT BUILD_SHARED_LIBS)
#   check_c_compiler_flag (-fPIC HAS_FPIC)
#   if (HAS_FPIC)
#     set (CMAKE_C_FLAGS "-fPIC ${CMAKE_C_FLAGS}")
#     set (CMAKE_CXX_FLAGS "-fPIC ${CMAKE_CXX_FLAGS}")
#   endif ()
# endif ()

install ( TARGETS opt
          EXPORT  OptPPLibraryDepends
          RUNTIME DESTINATION ${RELATIVE_INSTALL_BIN_DIR}
          LIBRARY DESTINATION ${RELATIVE_INSTALL_LIB_DIR}
          ARCHIVE DESTINATION ${RELATIVE_INSTALL_LIB_DIR}
        )


if (OPTPP_TESTS)
  enable_testing ()

  macro (optpp_add_test)
    add_executable(${ARGN})
    target_include_directories (${ARGV0} PRIVATE newmat11 include ${PROJECT_BINARY_DIR}/include)
    target_link_libraries(${ARGV0} opt)
    add_test(NAME ${ARGV0} COMMAND ${ARGV0})
  endmacro ()

  
  
  optpp_add_test(tstqnewton tests/uncon/tstqnewton.C tests/uncon/rosen.C)

  optpp_add_test(tstgnewton tests/uncon/tstgnewton.C tests/uncon/rosen.C)

  optpp_add_test(tstnewton tests/uncon/tstnewton.C tests/uncon/rosen.C)

  optpp_add_test(tstfdnlf1 tests/uncon/tstfdnlf1.C tests/uncon/rosen.C)

  optpp_add_test(tstcg tests/uncon/tstcg.C tests/uncon/rosen.C)

  optpp_add_test(tstLBFGS tests/uncon/tstLBFGS.C tests/uncon/rosen.C)

endif ()

set (CPACK_PACKAGE_NAME          "${CMAKE_PROJECT_NAME}")
set (CPACK_PACKAGE_VERSION_MAJOR "${OPTPP_MAJOR_VERSION}")
set (CPACK_PACKAGE_VERSION_MINOR "${OPTPP_MINOR_VERSION}")
set (CPACK_PACKAGE_VERSION_PATCH "${OPTPP_BUGFIX_VERSION}")
set (CPACK_SOURCE_GENERATOR      "TBZ2;TGZ"    )
set (CPACK_BINARY_STGZ           "OFF"        )
set (CPACK_BINARY_TBZ2           "ON"         )
set (CPACK_BINARY_TGZ            "ON"         )
set (CPACK_BINARY_TZ             "OFF"        )
set (CPACK_SOURCE_IGNORE_FILES ".git;/build;.*~;${CPACK_SOURCE_IGNORE_FILES}")

set (CPACK_SOURCE_PACKAGE_FILE_NAME ${CPACK_PACKAGE_NAME}-${OPTPP_VERSION_STRING})

include (CPack)

# # configuration files
# export (TARGETS ${nlopt_lib} NAMESPACE OptPP:: FILE ${PROJECT_BINARY_DIR}/OptPPLibraryDepends.cmake)
# 
# # Install the export set for use with the install-tree
# install(EXPORT OptPPLibraryDepends
#         NAMESPACE OptPP::
#         DESTINATION ${RELATIVE_INSTALL_CMAKE_DIR}
#         COMPONENT Development)
# 
# # Create a OPTPPConfig.cmake file for the use from the install tree
# # and install it
# set (OPTPP_LIBRARIES "OptPP::${nlopt_lib}")
# 
# set (OPTPP_CMAKE_DIR "${INSTALL_CMAKE_DIR}")
# file (RELATIVE_PATH rel_include_dir "${OPTPP_CMAKE_DIR}" "${INSTALL_INCLUDE_DIR}")
# list (APPEND RELATIVE_OPTPP_INCLUDE_DIRS ${rel_include_dir})
# 
# file (RELATIVE_PATH rel_lib_dir "${OPTPP_CMAKE_DIR}" "${INSTALL_LIB_DIR}")
# list (APPEND RELATIVE_OPTPP_LIB_DIR ${rel_lib_dir})
# 
# configure_file (cmake/OptPPConfig.cmake.in OptPPConfig.cmake @ONLY)
# configure_file (cmake/OptPPConfigVersion.cmake.in OptPPConfigVersion.cmake @ONLY)
# install (FILES
#           ${CMAKE_CURRENT_BINARY_DIR}/OptPPConfig.cmake
#           ${CMAKE_CURRENT_BINARY_DIR}/OptPPConfigVersion.cmake
#          DESTINATION ${RELATIVE_INSTALL_CMAKE_DIR}
#          COMPONENT Development)
