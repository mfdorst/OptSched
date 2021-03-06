cmake_minimum_required(VERSION 3.4.3)

# Exit if attempting to build as a standalone project.
IF (CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
  message(FATAL_ERROR "You cannot build opt-sched as a standalone project. Place opt-sched in the llvm tree under the \"projects\" directory.")
ENDIF()

IF (NOT CMAKE_BUILD_TYPE)
  message(FATAL_ERROR "Must set cmake build type.")
ENDIF ()

IF (${PACKAGE_VERSION} VERSION_LESS "7.0")
  add_definitions("-DLLVM_DEBUG=DEBUG")
ENDIF()

# Add OptSched debug defines eg: SET(OPT_SCHED_DEBUG_FLAGS "-DIS_DEBUG_DEFS_AND_USES -DIS_DEBUG_DEF_USE_COUNT")
SET(OPT_SCHED_DEBUG_FLAGS "")

# If asserts are enabled opt-sched must be built with "IS_DEBUG".
SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DIS_DEBUG")
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OPT_SCHED_DEBUG_FLAGS}")

IF (CMAKE_BUILD_TYPE EQUAL "DEBUG")
  set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
ENDIF ()

# OptSched sources
SET(OPT_SCHED_SRCS Scheduler/aco.cpp
  Scheduler/bb_spill.cpp
  Scheduler/buffers.cpp
  Scheduler/config.cpp
  Scheduler/data_dep.cpp
  Scheduler/enumerator.cpp
  Scheduler/gen_sched.cpp
  Scheduler/graph.cpp
  Scheduler/graph_trans.cpp
  Scheduler/hist_table.cpp
  Scheduler/list_sched.cpp
  Scheduler/logger.cpp
  Scheduler/reg_alloc.cpp
  Scheduler/utilities.cpp
  Scheduler/machine_model.cpp
  Scheduler/random.cpp
  Scheduler/ready_list.cpp
  Scheduler/register.cpp
  Scheduler/relaxed_sched.cpp
  Scheduler/sched_basic_data.cpp
  Scheduler/sched_region.cpp
  Scheduler/stats.cpp
  Wrapper/OptimizingScheduler.cpp
  Wrapper/OptSchedMachineWrapper.cpp
  Wrapper/OptSchedDDGWrapperBasic.cpp
  Wrapper/OptSchedGenericTarget.cpp
  Wrapper/TwoPassScheduler.cpp)

SET(OPT_SCHED_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/include)
SET (OPT_SCHED_TARGET_DEPS "LLVMCodeGen")

function(check_if_AMDGPU_supported)
  IF (${PACKAGE_VERSION} VERSION_LESS "7.0")
    message(WARNING "OptSched requries LLVM version >= 7.0 to build the AMDGPU scheduler.")
    SET(AMDGPU_SUPPORTED False PARENT_SCOPE)

  ELSE ()
    SET(AMDGPU_SUPPORTED True PARENT_SCOPE)

  ENDIF ()
endfunction(check_if_AMDGPU_supported)

# Only build and include AMDGPU functionality if it is being targeted.
IF ("AMDGPU" IN_LIST LLVM_TARGETS_TO_BUILD)
  check_if_AMDGPU_supported()
  IF (${AMDGPU_SUPPORTED})
    SET (OPT_SCHED_TARGET_DEPS ${OPT_SCHED_TARGET_DEPS} AMDGPUCommonTableGen)
    SET (OPT_SCHED_SRCS ${OPT_SCHED_SRCS} Wrapper/AMDGPU/GCNOptSched.cpp
                                          Wrapper/AMDGPU/OptSchedGCNTarget.cpp
                                          Wrapper/AMDGPU/OptSchedDDGWrapperGCN.cpp)
    SET (OPT_SCHED_INCLUDE_DIRS ${OPT_SCHED_INCLUDE_DIRS} ${CMAKE_CURRENT_SOURCE_DIR}/lib
                                ${LLVM_MAIN_SRC_DIR}/lib/Target/AMDGPU
                                ${CMAKE_CURRENT_BINARY_DIR}/../../lib/Target/AMDGPU)
  ENDIF ()
ENDIF ()

include_directories(${OPT_SCHED_INCLUDE_DIRS})
add_subdirectory(lib)
