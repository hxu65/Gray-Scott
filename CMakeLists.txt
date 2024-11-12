#------------------------------------------------------------------------------#
# Distributed under the OSI-approved Apache License, Version 2.0.  See
# accompanying file Copyright.txt for details.
#------------------------------------------------------------------------------#
# Define the BINARY_DIR variable
cmake_minimum_required(VERSION 3.22)
project(coeus)
find_package(ADIOS2 REQUIRED)
message(STATUS "Found Adios2")
MESSAGE(STATUS "Adios config: ${ADIOS2_CONFIG}")
# MPI
find_package(MPI REQUIRED COMPONENTS C CXX)
find_package(OpenMP REQUIRED COMPONENTS C CXX)
message(STATUS "Found OpenMP")
find_package(MPI REQUIRED COMPONENTS C CXX)
#spdlog
add_subdirectory(external_libraries/spdlog)
add_subdirectory(external_libraries/cereal)
add_subdirectory(external_libraries/rapidjson)

set(BINARY_DIR ${CMAKE_SOURCE_DIR})

# Pass the BINARY_DIR as a preprocessor definition
add_definitions(-DBINARY_DIR="${BINARY_DIR}")


add_executable(adios2-gray-scott
  simulation/main.cpp
  simulation/gray-scott.cpp
  simulation/settings.cpp
  simulation/writer.cpp
  simulation/restart.cpp
)
target_link_libraries(adios2-gray-scott adios2::adios2
         MPI::MPI_C)
target_include_directories(adios2-gray-scott PRIVATE

        ${ADIOS2_INCLUDE_DIR}
        ${CMAKE_SOURCE_DIR}/external_libraries/spdlog/include
        )
install(TARGETS adios2-gray-scott 
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

add_executable(adios2-pdf-calc analysis/pdf-calc.cpp)
target_link_libraries(adios2-pdf-calc
        adios2::adios2 hermes_engine  MPI::MPI_C)
target_include_directories(adios2-pdf-calc PRIVATE

        ${ADIOS2_INCLUDE_DIR}
        ${CMAKE_SOURCE_DIR}/external_libraries/spdlog/include
        )
install(TARGETS adios2-pdf-calc 
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})


add_executable(adios2-hashing analysis/hashing.cpp)

# Link libraries to the executable
target_link_libraries(adios2-hashing
        adios2::adios2
        MPI::MPI_C
        )

# Include necessary directories for compilation
target_include_directories(adios2-hashing PRIVATE
        ${CMAKE_SOURCE_DIR}/external_libraries/spdlog/include
        ${ADIOS2_INCLUDE_DIR}
        )

# Install the executable
install(TARGETS adios2-hashing
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        )

add_executable(hashing-comparison analysis/compare.cpp)

# Link libraries to the executable
target_link_libraries(hashing-comparison
        adios2::adios2
        MPI::MPI_C
        )

# Include necessary directories for compilation
target_include_directories(hashing-comparison PRIVATE
        ${ADIOS2_INCLUDE_DIR}
        ${CMAKE_SOURCE_DIR}/external_libraries/spdlog/include
        )

# Install the executable
install(TARGETS hashing-comparison
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        )



install(FILES "adios2.xml" "adios2-inline-plugin.xml" "adios2-fides-staging.xml"
                           "visit-bp4.session" "visit-bp4.session.gui" 
                           "visit-sst.session" "visit-sst.session.gui"                  
                           "simulation/settings-files.json"
                           "simulation/settings-staging.json"
                           "simulation/settings-inline.json"
                           "plot/decomp.py" "plot/gsplot.py" "plot/pdfplot.py"
                           "README.md" 
        DESTINATION ${CMAKE_INSTALL_PREFIX}/share/adios2-examples/gray-scott) 

install(DIRECTORY "catalyst"
        DESTINATION ${CMAKE_INSTALL_PREFIX}/share/adios2-examples/gray-scott) 


install(PROGRAMS "cleanup.sh" 
        DESTINATION ${CMAKE_INSTALL_PREFIX}/share/adios2-examples/gray-scott) 

