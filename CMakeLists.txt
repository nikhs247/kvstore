cmake_minimum_required(VERSION 3.15)
project(kvstore CXX)

# Setup C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Find dependencies
find_package(Protobuf REQUIRED)
find_package(gRPC CONFIG REQUIRED)

# Define the proto file path
set(PROTO_FILE "proto/raft.proto")

# Generate C++ sources from the .proto file
# Invoke protoc compiler during the build
add_library(
    raft_proto_lib 
    SHARED
    ${PROTO_FILE}
)

target_link_libraries(
    raft_proto_lib
    PUBLIC
    protobuf::libprotobuf
    gRPC::grpc++    
)

# Fetch the gRPC Plugin Location
get_target_property(
    gRPC_CPP_PLUGIN_EXECUTABLE gRPC::grpc_cpp_plugin 
    IMPORTED_LOCATION
)

# Generate Standard Protobuf files
protobuf_generate(
    TARGET raft_proto_lib 
    LANGUAGE cpp
)

# Generate gRPC Service files
protobuf_generate(
    TARGET raft_proto_lib 
    LANGUAGE grpc
    PLUGIN "protoc-gen-grpc=${gRPC_CPP_PLUGIN_EXECUTABLE}"
    GENERATE_EXTENSIONS ".grpc.pb.h" ".grpc.pb.cc"
)

# Define server executable
add_executable(
    raft_server
    src/main.cpp
    src/raft_node.cpp
    src/kv_store.cpp
)

# Link everything
target_link_libraries(
    raft_server
    PRIVATE
    raft_proto_lib
    gRPC::grpc++
    gRPC::grpc++_reflection
)

# Include directories
target_include_directories(
    raft_server
    PRIVATE
    ${CMAKE_CURRENT_BINARY_DIR}
    include
)