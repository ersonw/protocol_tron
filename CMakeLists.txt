#find_package(Protobuf CONFIG REQUIRED)
find_package(Protobuf)
find_package(gRPC CONFIG REQUIRED)

set(_PROTOBUF_LIBPROTOBUF protobuf::libprotobuf)
set(_REFLECTION gRPC::grpc++_reflection)
set(_GRPC_GRPCPP gRPC::grpc++)
SET(PROTO_ROOT ${CMAKE_CURRENT_LIST_DIR})

include_directories(${Protobuf_INCLUDE_DIRS} ${gRPC_INCLUDE_DIRS})
find_program(PROTOBUF_EXEC protoc)

find_program(gRPC_CPP_PLUGIN_EXECUTABLE grpc_cpp_plugin)


FILE(GLOB_RECURSE PROTO_INPUT_GOOGLE google/*.proto core/*.proto)
FILE(GLOB_RECURSE PROTO_INPUT api/*.proto)
SET(PROTOC ${PROTOBUF_EXEC})
SET(TRON_PROTOBUF ${CMAKE_CURRENT_BINARY_DIR})


FOREACH(PROTO_FILE ${PROTO_INPUT_GOOGLE})
    string(REPLACE "${CMAKE_CURRENT_LIST_DIR}" "" RELATIVE_PATH "${PROTO_FILE}")
    GET_FILENAME_COMPONENT(PROTO_NAME ${RELATIVE_PATH} NAME_WE)
    GET_FILENAME_COMPONENT(DIRECTORY_NAME ${RELATIVE_PATH} DIRECTORY)

    SET(CUR_PROTO_GEN
            ${TRON_PROTOBUF}${DIRECTORY_NAME}/${PROTO_NAME}.pb.h
            ${TRON_PROTOBUF}${DIRECTORY_NAME}/${PROTO_NAME}.pb.cc
    )
    #    message(WARNING ${CUR_PROTO_GEN})
    SET(PROTO_GEN_GOOGLE_FILES
            ${PROTO_GEN_GOOGLE_FILES}
            ${CUR_PROTO_GEN}
    )
    ADD_CUSTOM_COMMAND(
            OUTPUT ${CUR_PROTO_GEN}
            COMMAND ${PROTOC}  --cpp_out=${TRON_PROTOBUF} -I${CMAKE_CURRENT_LIST_DIR} ${PROTO_FILE}
            DEPENDS ${PROTOC} ${PROTO_FILE}
            WORKING_DIRECTORY ${PROTO_ROOT}
    )
endforeach(PROTO_FILE ${PROTO_INPUT_GOOGLE})
FOREACH(PROTO_FILE ${PROTO_INPUT})
    string(REPLACE "${CMAKE_CURRENT_LIST_DIR}" "" RELATIVE_PATH "${PROTO_FILE}")
    GET_FILENAME_COMPONENT(PROTO_NAME ${RELATIVE_PATH} NAME_WE)
    GET_FILENAME_COMPONENT(DIRECTORY_NAME ${RELATIVE_PATH} DIRECTORY)

    SET(CUR_PROTO_GEN
            ${TRON_PROTOBUF}${DIRECTORY_NAME}/${PROTO_NAME}.pb.h
            ${TRON_PROTOBUF}${DIRECTORY_NAME}/${PROTO_NAME}.pb.cc
    )
    #    message(WARNING ${CUR_PROTO_GEN})
    SET(PROTO_GEN_FILES
            ${PROTO_GEN_FILES}
            ${CUR_PROTO_GEN}
    )
    ADD_CUSTOM_COMMAND(
            OUTPUT ${CUR_PROTO_GEN}
            COMMAND ${PROTOC}  --cpp_out=${TRON_PROTOBUF} -I${CMAKE_CURRENT_LIST_DIR} ${PROTO_FILE}
            DEPENDS ${PROTOC} ${PROTO_FILE}
            WORKING_DIRECTORY ${PROTO_ROOT}
    )

    SET(GRPC_CUR_PROTO_GEN
            ${TRON_PROTOBUF}${DIRECTORY_NAME}/${PROTO_NAME}.grpc.pb.h
            ${TRON_PROTOBUF}${DIRECTORY_NAME}/${PROTO_NAME}.grpc.pb.cc)


    SET(GRPC_PROTO_GEN_FILES
            ${GRPC_PROTO_GEN_FILES}
            ${GRPC_CUR_PROTO_GEN}
    )

    GET_FILENAME_COMPONENT(absolute_proto ${PROTO_FILE} ABSOLUTE)
    ADD_CUSTOM_COMMAND(
            OUTPUT ${GRPC_CUR_PROTO_GEN}
            COMMAND ${PROTOC}
            ARGS --grpc_out="${TRON_PROTOBUF}" -I "${CMAKE_CURRENT_LIST_DIR}"
            --plugin=protoc-gen-grpc="${gRPC_CPP_PLUGIN_EXECUTABLE}" "${absolute_proto}"
            DEPENDS "${absolute_proto}"
            WORKING_DIRECTORY ${PROTO_ROOT}
    )
endforeach(PROTO_FILE ${PROTO_INPUT})

#add_custom_target(generate_grpc ALL
#        DEPENDS ${PROTO_GEN_FILES} ${GRPC_PROTO_GEN_FILES}
#        COMMENT "generate gRPC target"
#        VERBATIM
#)

add_library(grpc_proto_gen STATIC ${PROTO_GEN_FILES} ${GRPC_PROTO_GEN_FILES} ${PROTO_GEN_GOOGLE_FILES})
target_include_directories(grpc_proto_gen PUBLIC ${PROTO_ROOT} ${TRON_PROTOBUF} )

#message(WARNING ${_PROTOBUF_LIBPROTOBUF})
target_link_libraries(grpc_proto_gen
        ${Protobuf_LIBPROTOBUF}
        ${_REFLECTION}
        ${_GRPC_GRPCPP}
)