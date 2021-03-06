cmake_minimum_required(VERSION 3.16)
project(integer_parsing)

option(QND_TEST_BOOST "Include Boost::Spirit::Qi in benchmarks" OFF)

find_package(GTest REQUIRED)
if (${QND_TEST_BOOST})
  find_package(Boost REQUIRED)
endif()

set(CMAKE_CXX_STANDARD 20)

add_library(
  parsing_lib
  parsing.cc
)
set(EXTRA_FLAGS -O3 -g -march=native -mtune=native -Wall -Werror)
target_compile_options(parsing_lib PUBLIC ${EXTRA_FLAGS})
if (${QND_TEST_BOOST})
  target_compile_options(parsing_lib PUBLIC -DQND_TEST_BOOST)
  target_link_libraries(parsing_lib PUBLIC Boost::boost)
endif()

add_executable(
  main
  main.cc
)
target_link_libraries(main PRIVATE parsing_lib)

find_package(benchmark REQUIRED)
add_executable(
  bench
  bench.cc
)
target_link_libraries(bench PRIVATE parsing_lib benchmark::benchmark)

SET(BENCHMARK_OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/benchmark-results.json)
add_custom_target(
  run-benchmark
  BYPRODUCTS ${BENCHMARK_OUTPUT}
  COMMAND ${CMAKE_CURRENT_BINARY_DIR}/bench
    --benchmark_out=${BENCHMARK_OUTPUT}
    --benchmark_out_format=json --benchmark_repetitions=20
  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
)
add_dependencies(run-benchmark bench)


add_custom_target(disable-cpu-frequency-scaling)
add_custom_command(
  TARGET disable-cpu-frequency-scaling
  COMMAND sudo cpupower frequency-set -g performance
)

add_executable(
  test
  test/test.cc
)
target_link_libraries(test PRIVATE parsing_lib GTest::Main)
