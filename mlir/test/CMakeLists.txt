add_subdirectory(CAPI)
add_subdirectory(lib)

if(MLIR_ENABLE_BINDINGS_PYTHON)
  add_subdirectory(python)
endif()

llvm_canonicalize_cmake_booleans(
  MLIR_ENABLE_BINDINGS_PYTHON
  LLVM_BUILD_EXAMPLES
  MLIR_ENABLE_CUDA_CONVERSIONS
  MLIR_ENABLE_CUDA_RUNNER
  MLIR_ENABLE_ROCM_CONVERSIONS
  MLIR_ENABLE_ROCM_RUNNER
  MLIR_ENABLE_SPIRV_CPU_RUNNER
  MLIR_ENABLE_VULKAN_RUNNER
  )

# Passed to lit.site.cfg.py.so that the out of tree Standalone dialect test
# can find MLIR's CMake configuration
set(MLIR_CMAKE_DIR
  "${CMAKE_BINARY_DIR}/lib${LLVM_LIBDIR_SUFFIX}/cmake/mlir")

# Passed to lit.site.cfg.py.in to set up the path where to find the libraries
# for linalg integration tests.
set(MLIR_DIALECT_LINALG_INTEGRATION_TEST_LIB_DIR ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})
set(MLIR_RUNNER_UTILS_DIR ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})

# Passed to lit.site.cfg.py.in to set up the path where to find the libraries
# for the mlir spirv / vulkan runner tests.
set(MLIR_SPIRV_WRAPPER_LIBRARY_DIR ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})
set(MLIR_VULKAN_WRAPPER_LIBRARY_DIR ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})

if (MLIR_INCLUDE_INTEGRATION_TESTS)
  set(INTEL_SDE_EXECUTABLE "" CACHE STRING
      "If set, arch-specific integration tests are run with Intel SDE.")
  option(MLIR_RUN_AMX_TESTS "Run AMX tests.")
  option(MLIR_RUN_X86VECTOR_TESTS "Run X86Vector tests.")
  option(MLIR_RUN_CUDA_TENSOR_CORE_TESTS "Run CUDA Tensor core WMMA tests.")
  # Passed to lit.site.cfg.py.in to set up the path where to find the libraries.
  set(MLIR_INTEGRATION_TEST_DIR ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})

  # Copy test data over.
  file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/Integration/data/mttkrp_b.tns
            ${CMAKE_CURRENT_SOURCE_DIR}/Integration/data/test.mtx
            ${CMAKE_CURRENT_SOURCE_DIR}/Integration/data/test.tns
            ${CMAKE_CURRENT_SOURCE_DIR}/Integration/data/wide.mtx
            ${CMAKE_CURRENT_SOURCE_DIR}/Integration/data/zero.mtx
          DESTINATION ${MLIR_INTEGRATION_TEST_DIR}/data/)
endif()

configure_lit_site_cfg(
  ${CMAKE_CURRENT_SOURCE_DIR}/lit.site.cfg.py.in
  ${CMAKE_CURRENT_BINARY_DIR}/lit.site.cfg.py
  MAIN_CONFIG
  ${CMAKE_CURRENT_SOURCE_DIR}/lit.cfg.py
  )
configure_lit_site_cfg(
  ${CMAKE_CURRENT_SOURCE_DIR}/Unit/lit.site.cfg.py.in
  ${CMAKE_CURRENT_BINARY_DIR}/Unit/lit.site.cfg.py
  MAIN_CONFIG
  ${CMAKE_CURRENT_SOURCE_DIR}/Unit/lit.cfg.py
  )

set(MLIR_TEST_DEPENDS
  FileCheck count not
  mlir-capi-execution-engine-test
  mlir-capi-ir-test
  mlir-capi-llvm-test
  mlir-capi-pass-test
  mlir-capi-sparse-tensor-test
  mlir-cpu-runner
  mlir-linalg-ods-gen
  mlir-linalg-ods-yaml-gen
  mlir-lsp-server
  mlir-opt
  mlir-reduce
  mlir-tblgen
  mlir-translate
  mlir_runner_utils
  mlir_c_runner_utils
  mlir_async_runtime
  )

if (MLIR_INCLUDE_INTEGRATION_TESTS)
  list(APPEND MLIR_TEST_DEPENDS lli)
endif()

if(MLIR_ENABLE_CUDA_RUNNER)
  list(APPEND MLIR_TEST_DEPENDS mlir_cuda_runtime)
endif()

if(MLIR_ENABLE_ROCM_RUNNER)
  list(APPEND MLIR_TEST_DEPENDS mlir_rocm_runtime)
endif()

list(APPEND MLIR_TEST_DEPENDS MLIRUnitTests)

if(LLVM_BUILD_EXAMPLES)
  list(APPEND MLIR_TEST_DEPENDS
    toyc-ch1
    toyc-ch2
    toyc-ch3
    toyc-ch4
    toyc-ch5
    toyc-ch6
    toyc-ch7
    )
endif()

if(MLIR_ENABLE_SPIRV_CPU_RUNNER)
  add_subdirectory(mlir-spirv-cpu-runner)
  list(APPEND MLIR_TEST_DEPENDS
    mlir-spirv-cpu-runner
    mlir_test_spirv_cpu_runner_c_wrappers
  )
endif()

if(MLIR_ENABLE_VULKAN_RUNNER)
  list(APPEND MLIR_TEST_DEPENDS
    mlir-vulkan-runner
  )
endif()

if(MLIR_ENABLE_BINDINGS_PYTHON)
  list(APPEND MLIR_TEST_DEPENDS
    MLIRBindingsPythonExtension
    MLIRBindingsPythonSources
    MLIRBindingsPythonTestOps
    MLIRTransformsBindingsPythonExtension
    MLIRConversionsBindingsPythonExtension
  )
endif()

add_lit_testsuite(check-mlir "Running the MLIR regression tests"
  ${CMAKE_CURRENT_BINARY_DIR}
  DEPENDS ${MLIR_TEST_DEPENDS}
  )
set_target_properties(check-mlir PROPERTIES FOLDER "Tests")

add_lit_testsuites(MLIR ${CMAKE_CURRENT_SOURCE_DIR}
  DEPENDS ${MLIR_TEST_DEPENDS}
)
