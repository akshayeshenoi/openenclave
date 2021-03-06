# Copyright (c) Open Enclave SDK contributors.
# Licensed under the MIT License.

cmake_minimum_required(VERSION 3.11)

project("Attestation Sample" LANGUAGES C CXX)

find_package(OpenEnclave CONFIG REQUIRED)

set(CMAKE_CXX_STANDARD 11)
# set OE_CRYPTO_LIB to either "mbedtls" or "openssl" based on the crypto wrapper to be used.
# OE_CRYPTO_LIB is case sensitive. Use all lowercase letters.
set(OE_CRYPTO_LIB mbedtls)

add_subdirectory(common)
add_subdirectory(enclave_a)
add_subdirectory(enclave_b)
add_subdirectory(host)

add_custom_target(sign ALL DEPENDS enclave_a_signed enclave_b_signed)

if ((NOT DEFINED ENV{OE_SIMULATION}) OR (NOT $ENV{OE_SIMULATION}))
  add_custom_target(run DEPENDS runsgxlocal runsgxremote)

  add_custom_target(
    runsgxlocal
    DEPENDS attestation_host sign
    COMMAND
      attestation_host sgxlocal ${CMAKE_BINARY_DIR}/enclave_a/enclave_a.signed
      ${CMAKE_BINARY_DIR}/enclave_b/enclave_b.signed)

  add_custom_target(
    runsgxremote
    DEPENDS attestation_host sign
    COMMAND
      attestation_host sgxremote ${CMAKE_BINARY_DIR}/enclave_a/enclave_a.signed
      ${CMAKE_BINARY_DIR}/enclave_b/enclave_b.signed)
endif ()
