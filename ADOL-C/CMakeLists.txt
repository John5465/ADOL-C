add_subdirectory(include)
add_subdirectory(src)
add_subdirectory(doc)

set(ADOLC_INCLUDE_DIR "${CMAKE_BINARY_DIR}/install/include")

add_executable(speelpenning
    examples/speelpenning.cpp
)
target_include_directories(speelpenning PRIVATE
    ${ADOLC_INCLUDE_DIR}
)
target_link_directories(speelpenning PRIVATE
    "${CMAKE_BINARY_DIR}/Debug"
)
target_link_libraries(speelpenning
    adolc.lib
)

add_executable(powexam
    examples/powexam.cpp
)
target_include_directories(powexam PRIVATE
    ${ADOLC_INCLUDE_DIR}
)
target_link_directories(powexam PRIVATE
    "${CMAKE_BINARY_DIR}/Debug"
)
target_link_libraries(powexam
    adolc.lib
)