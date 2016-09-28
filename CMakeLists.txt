cmake_minimum_required(VERSION 2.8)

project( open62541_namespaces )


SET(UA_ENABLE_GENERATE_NAMESPACE0 ON FORCE)
set(UA_ENABLE_AMALGAMATION OFF CACHE BOOL "Use Amalgamation" FORCE)

# open62541 dependency
if(UA_ENABLE_AMALGAMATION)
    include_directories(${PROJECT_BINARY_DIR}/deps/open62541) # contains open62541.h
else()
    add_definitions(-DUA_NO_AMALGAMATION)
    include_directories(${PROJECT_BINARY_DIR}/deps/open62541/src_generated)
    include_directories(${PROJECT_SOURCE_DIR}/deps/open62541/include)
    include_directories(${PROJECT_SOURCE_DIR}/deps/open62541/plugins)
    include_directories(${PROJECT_SOURCE_DIR}/deps/open62541/src)
    include_directories(${PROJECT_SOURCE_DIR}/deps/open62541/deps)
endif()
add_subdirectory(deps/open62541)
include_directories(${CMAKE_BINARY_DIR}/deps/open62541)
link_directories(${CMAKE_BINARY_DIR}/deps/open62541)


include_directories(${PROJECT_BINARY_DIR}/src_generated)
file(MAKE_DIRECTORY ${PROJECT_BINARY_DIR}/src_generated)

# generated namespace for custom .xml models
add_custom_command(OUTPUT ${PROJECT_BINARY_DIR}/src_generated/test_generated.c
        ${PROJECT_BINARY_DIR}/src_generated/test_generated.h
        PRE_BUILD
        COMMAND ${PYTHON_EXECUTABLE} ${PROJECT_SOURCE_DIR}/deps/open62541/tools/pyUANamespace/generate_open62541CCode.py
        -i ${PROJECT_SOURCE_DIR}/deps/open62541/tools/pyUANamespace/NodeID_Blacklist_FullNS0.txt
        ${PROJECT_SOURCE_DIR}/deps/open62541/tools/schema/namespace0/Opc.Ua.NodeSet2.xml
        ${PROJECT_SOURCE_DIR}/model/Opc.Ua.Di.NodeSet2.xml
        ${PROJECT_SOURCE_DIR}/model/project1.NodeSet2.xml
        ${PROJECT_SOURCE_DIR}/model/project2.NodeSet2.xml
        ${PROJECT_BINARY_DIR}/src_generated/test_generated
        DEPENDS ${PROJECT_SOURCE_DIR}/deps/open62541/tools/schema/namespace0/Opc.Ua.NodeSet2.xml
        ${PROJECT_SOURCE_DIR}/model/Opc.Ua.Di.NodeSet2.xml
        ${PROJECT_SOURCE_DIR}/model/project1.NodeSet2.xml
        ${PROJECT_SOURCE_DIR}/model/project2.NodeSet2.xml
        ${PROJECT_SOURCE_DIR}/deps/open62541/tools/pyUANamespace/generate_open62541CCode.py)

set(HDRS ${PROJECT_BINARY_DIR}/src_generated/test_generated.h
        )

set(SRCS ${PROJECT_BINARY_DIR}/src_generated/test_generated.c
        )

add_executable( example ${SRCS} src/main.c)
target_link_libraries(example open62541 )
add_dependencies( example open62541 )