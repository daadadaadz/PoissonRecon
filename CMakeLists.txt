cmake_minimum_required(VERSION 3.16)

project(PoissonRecon VERSION 1.0 LANGUAGES CXX)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

option(POISSON_FAST_COMPILE "Build only the default template instantiation set" OFF)
option(POISSON_BIG_DATA "Enable BIG_DATA mode for larger indexing ranges" OFF)
option(POISSON_SANITIZED_PR "Enable sanitizers defined in PreProcessor.h" OFF)
option(POISSON_SHOW_WARNINGS "Display compilation warnings" OFF)
option(POISSON_ARRAY_DEBUG "Enable array bounds debugging checks" OFF)

find_package(OpenMP)
find_package(ZLIB REQUIRED)
find_package(PNG REQUIRED)
find_package(JPEG REQUIRED)

set(core_headers
    src/core/Allocator.h
    src/core/Array.h
    src/core/Array.inl
    src/core/BinaryNode.h
    src/core/Factor.h
    src/core/Geometry.h
    src/core/Geometry.inl
    src/core/MyAtomic.h
    src/core/MyExceptions.h
    src/core/MyMiscellany.h
    src/core/NestedVector.h
    src/core/PPolynomial.h
    src/core/PPolynomial.inl
    src/core/ParameterPack.h
    src/core/Polynomial.h
    src/core/Polynomial.inl
    src/core/PreProcessor.h
    src/core/Streams.h
)

set(io_headers
    src/io/CmdLineParser.h
    src/io/CmdLineParser.inl
    src/io/DataStream.h
    src/io/DataStream.imp.h
    src/io/DataStream.imp.inl
    src/io/DataStream.inl
    src/io/Image.h
    src/io/JPEG.h
    src/io/JPEG.inl
    src/io/PNG.h
    src/io/PNG.inl
    src/io/Ply.h
    src/io/Ply.inl
    src/io/PlyFile.h
    src/io/PlyFile.inl
)

set(linalg_headers
    src/linalg/LinearSolvers.h
    src/linalg/MAT.h
    src/linalg/MAT.inl
    src/linalg/SparseMatrix.h
    src/linalg/SparseMatrix.inl
    src/linalg/SparseMatrixInterface.h
    src/linalg/SparseMatrixInterface.inl
)

set(parallel_headers
    src/parallel/MultiThreading.h
)

set(recon_headers
    src/recon/BSplineData.h
    src/recon/BSplineData.inl
    src/recon/FEMTree.Evaluation.inl
    src/recon/FEMTree.Initialize.inl
    src/recon/FEMTree.LevelSet.2D.inl
    src/recon/FEMTree.LevelSet.3D.inl
    src/recon/FEMTree.LevelSet.inl
    src/recon/FEMTree.SortedTreeNodes.inl
    src/recon/FEMTree.System.inl
    src/recon/FEMTree.WeightedSamples.inl
    src/recon/FEMTree.h
    src/recon/FEMTree.inl
    src/recon/MarchingCubes.h
    src/recon/PointExtent.h
    src/recon/PointExtent.inl
    src/recon/Rasterizer.h
    src/recon/Rasterizer.inl
    src/recon/Reconstructors.h
    src/recon/Reconstructors.streams.h
    src/recon/RegularGrid.h
    src/recon/RegularGrid.inl
    src/recon/RegularTree.h
    src/recon/RegularTree.inl
    src/recon/VertexFactory.h
    src/recon/VertexFactory.inl
    src/recon/Window.h
    src/recon/Window.inl
)

set(poisson_headers
    ${core_headers}
    ${io_headers}
    ${linalg_headers}
    ${parallel_headers}
    ${recon_headers}
)

add_library(poisson_core INTERFACE)
add_library(PoissonRecon::poisson_core ALIAS poisson_core)

target_include_directories(poisson_core
    INTERFACE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/core>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/io>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/linalg>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/parallel>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/recon>
        $<INSTALL_INTERFACE:include>
)
target_compile_features(poisson_core INTERFACE cxx_std_17)

if(POISSON_FAST_COMPILE)
    target_compile_definitions(poisson_core INTERFACE FAST_COMPILE)
endif()
if(POISSON_BIG_DATA)
    target_compile_definitions(poisson_core INTERFACE BIG_DATA)
endif()
if(POISSON_SANITIZED_PR)
    target_compile_definitions(poisson_core INTERFACE SANITIZED_PR)
endif()
if(POISSON_SHOW_WARNINGS)
    target_compile_definitions(poisson_core INTERFACE SHOW_WARNINGS)
endif()
if(POISSON_ARRAY_DEBUG)
    target_compile_definitions(poisson_core INTERFACE ARRAY_DEBUG)
endif()

if(OpenMP_CXX_FOUND)
    target_link_libraries(poisson_core INTERFACE OpenMP::OpenMP_CXX)
endif()
target_link_libraries(poisson_core INTERFACE PNG::PNG JPEG::JPEG ZLIB::ZLIB)

add_executable(poisson_recon src/recon/PoissonRecon.cpp)
target_link_libraries(poisson_recon PRIVATE poisson_core)

install(TARGETS poisson_core poisson_recon
    EXPORT PoissonReconTargets
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(DIRECTORY src/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(EXPORT PoissonReconTargets
    FILE PoissonReconTargets.cmake
    NAMESPACE PoissonRecon::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/PoissonRecon
)

set(POISSON_WITH_OPENMP ${OpenMP_CXX_FOUND})

configure_package_config_file(
    cmake/PoissonReconConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/PoissonReconConfig.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/PoissonRecon
)

write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/PoissonReconConfig-version.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

install(
    FILES
        ${CMAKE_CURRENT_BINARY_DIR}/PoissonReconConfig.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/PoissonReconConfig-version.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/PoissonRecon
)
