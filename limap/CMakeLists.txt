add_subdirectory(base)
add_subdirectory(ceresbase)
add_subdirectory(solvers)
add_subdirectory(vplib)
add_subdirectory(util)
add_subdirectory(pointsfm)
add_subdirectory(triangulation)
add_subdirectory(merging)
add_subdirectory(undistortion)
add_subdirectory(evaluation)
add_subdirectory(fitting)
add_subdirectory(estimators)
add_subdirectory(optimize)
add_subdirectory(structures)
if(INTERPOLATION_ENABLED)
    add_subdirectory(features)
endif()

add_subdirectory(_limap)

LIMAP_ADD_STATIC_LIBRARY(limap ${LIMAP_SOURCES})
if (NOT DEFINED CMAKE_DEBUG_POSTFIX)
    set(CMAKE_DEBUG_POSTFIX "d")
endif()
set_target_properties(limap PROPERTIES DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})
target_link_libraries(limap
    ${LIMAP_INTERNAL_LIBRARIES} ${LIMAP_EXTERNAL_LIBRARIES})

