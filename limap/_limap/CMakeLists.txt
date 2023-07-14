set(FOLDER_NAME "pylimap")

LIMAP_ADD_SOURCES(
    helpers.h
)

LIMAP_ADD_PYMODULE(_limap
    bindings.cc)

if (NOT DEFINED CMAKE_DEBUG_POSTFIX)
    set(CMAKE_DEBUG_POSTFIX "d")
endif()
set_target_properties(_limap PROPERTIES
    DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX})
target_link_libraries(_limap PRIVATE
    ${LIMAP_INTERNAL_LIBRARIES} ${LIMAP_EXTERNAL_LIBRARIES})


