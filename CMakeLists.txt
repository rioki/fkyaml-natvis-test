cmake_minimum_required(VERSION 3.21)
project(repro LANGUAGES CXX)

find_package(fkYAML CONFIG REQUIRED)

add_library(test STATIC test.cpp)
target_link_libraries(test PRIVATE fkYAML::fkYAML)