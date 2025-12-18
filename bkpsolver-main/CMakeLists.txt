cmake_minimum_required(VERSION 3.10)

project(BKPSolver)

add_compile_options(
    -march=native
    -mtune=native
    -mprefer-vector-width=512
)

# --- Gurobi 10.0 paths ---
set(GUROBI_HOME /Library/gurobi1000/macos_universal2)
include_directories(${GUROBI_HOME}/include)
link_directories(${GUROBI_HOME}/lib)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR})

# --- Include 3rdparty ---
include_directories(${CMAKE_SOURCE_DIR}/3rdparty/include)
link_directories(${CMAKE_SOURCE_DIR}/3rdparty/lib)

add_executable(bkpsolver
    src/bkp_instance.cpp
    src/dcs_solver.cpp
    src/main.cpp
)

target_link_libraries(bkpsolver
    knapsacksolver
    info
    greedy
    upper_bound_dantzig
    surrogate_relaxation
    dynamic_programming_primal_dual
    ${GUROBI_HOME}/lib/libgurobi_c++.a
    gurobi100
)

# --- OpenMP fix for Apple M2/Homebrew ---
find_package(OpenMP)

if(OpenMP_CXX_FOUND)
    target_link_libraries(bkpsolver OpenMP::OpenMP_CXX)
else()
    # Explicit Homebrew paths for Apple M2
    include_directories(/opt/homebrew/include)
    target_link_libraries(bkpsolver /opt/homebrew/lib/libomp.dylib)
    # Also set compiler flags so headers are found
    target_compile_options(bkpsolver PRIVATE -Xpreprocessor -fopenmp -I/opt/homebrew/include)
    target_link_options(bkpsolver PRIVATE -L/opt/homebrew/lib -lomp)
endif()

