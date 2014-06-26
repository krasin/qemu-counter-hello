cmake_minimum_required (VERSION 2.8.7)
project(qemu-counter-hello)

include(ExternalProject)

ExternalProject_Add(qemu
  PREFIX qemu
  GIT_REPOSITORY https://github.com/krasin/qemu-counter-hello
  CONFIGURE_COMMAND <SOURCE_DIR>/qemu/configure --target-list=arm-softmmu
  INSTALL_COMMAND ""
  )

ExternalProject_Add(arm-eabi-toolchain
  PREFIX arm-eabi
  URL https://sourcery.mentor.com/GNUToolchain/package12774/public/arm-none-eabi/arm-2014.05-28-arm-none-eabi-i686-pc-linux-gnu.tar.bz2
  URL_MD5 02fc7078af9c9c7e792f026c772374ae
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
  )

SET(CMAKE_SYSTEM_NAME Generic)
SET(CMAKE_SYSTEM_VERSION 1)
SET(CMAKE_C_COMPILER ${PROJECT_BINARY_DIR}/arm-eabi/src/arm-eabi-toolchain/bin/arm-none-eabi-gcc)
SET(CMAKE_CXX_COMPILER ${PROJECT_BINARY_DIR}/arm-eabi/src/arm-eabi-toolchain/bin/arm-none-eabi-g++)

SET(CMAKE_FIND_ROOT_PATH
  ${PROJECT_BINARY_DIR}/arm-eabi/src/arm-eabi-toolchain/arm-none-eabi/bin
  ${PROJECT_BINARY_DIR}/arm-eabi/src/arm-eabi-toolchain/arm-none-eabi/lib
  ${PROJECT_BINARY_DIR}/arm-eabi/src/arm-eabi-toolchain/arm-none-eabi/include
  )
SET(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM ONLY)
SET(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
SET(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)

SET(CMAKE_C_FLAGS "-g -mcpu=cortex-m3 -mthumb -T generic-m-hosted.ld -std=c99")
SET(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS)

add_executable(hello.elf hello.c)
add_dependencies(hello.elf arm-eabi-toolchain qemu)

SET(QEMU_EXECUTABLE ${PROJECT_BINARY_DIR}/qemu/src/qemu-build/arm-softmmu/qemu-system-arm)

add_custom_command(TARGET hello.elf
  COMMENT "Run hello.elf with qemu without counter device"
  COMMAND ${QEMU_EXECUTABLE} -machine versatilepb -cpu cortex-m3 -nographic
  -monitor null -serial null -semihosting -kernel hello.elf)

message("WARNING: the test with the counter device enabled is currently broken and thus disabled.")

#add_custom_command(TARGET hello.elf
#  COMMENT "Run hello.elf with qemu with counter device"
#  COMMAND ${QEMU_EXECUTABLE} -machine versatilepb -cpu cortex-m3 -nographic
#  -monitor null -serial null -semihosting -device read_counter -kernel hello.elf)
