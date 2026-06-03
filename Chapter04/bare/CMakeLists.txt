cmake_minimum_required(VERSION 3.13)

set(CMAKE_EXPORT_COMPILE_COMMANDS 1)

# Compiler programs and cross-compilation environment are configured
# in the toolchain file (gcc-arm-none-eabi.cmake).

set(RENODE "E:/Program Files/Renode/bin/renode.exe" CACHE STRING "Path to Renode executable")
message(STATUS "Using Renode at: ${RENODE}")


find_program(CCACHE_FOUND ccache)
if(CCACHE_FOUND)
  set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
endif(CCACHE_FOUND)

set(CDEFS "-DUSE_HAL_DRIVER -DSTM32F072xB")
set(MCU "-mcpu=cortex-m0 -mthumb")
set(COMMON_FLAGS "${MCU} ${CDEFS} -Wno-address-of-packed-member -Wextra -Wno-unused-parameter")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${COMMON_FLAGS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${COMMON_FLAGS} -Wno-register")
set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} ${COMMON_FLAGS} -x assembler-with-cpp")

set(CMAKE_C_FLAGS_DEBUG "-g -gdwarf-2 -O0")
set(CMAKE_CXX_FLAGS_DEBUG "-g -gdwarf-2 -O0")
set(CMAKE_C_FLAGS_RELEASE "-O2 -flto")
set(CMAKE_CXX_FLAGS_RELEASE "-O2 -flto")
set(CMAKE_C_FLAGS_MINSIZEREL "-Os -flto")
set(CMAKE_CXX_FLAGS_MINSIZEREL "-Os -flto")

if(CMAKE_EXPORT_COMPILE_COMMANDS)
  execute_process(
    COMMAND
      bash -c
      "${CMAKE_CXX_COMPILER} -x c++ -Wp,-v /dev/null 2>&1 > /dev/null | grep '^ /' | grep -w 'c++'"
    OUTPUT_VARIABLE COMPILER_HEADERS
    OUTPUT_STRIP_TRAILING_WHITESPACE)

  if(COMPILER_HEADERS)
    string(REGEX REPLACE "[[:space:]]+" ";" INCLUDE_COMPILER_HEADERS
           "${COMPILER_HEADERS}")
  endif()

  execute_process(
    COMMAND
      bash -c
      "${CMAKE_C_COMPILER} -x c -Wp,-v /dev/null 2>&1 > /dev/null | grep '^ /' "
    OUTPUT_VARIABLE COMPILER_HEADERS
    OUTPUT_STRIP_TRAILING_WHITESPACE)

  if(COMPILER_HEADERS)
    string(REGEX REPLACE "[[:space:]]+" ";" INCLUDE_COMPILER_HEADERS
           "${COMPILER_HEADERS}")
  endif()

  set(CMAKE_CXX_STANDARD_INCLUDE_DIRECTORIES
      "${CMAKE_CXX_STANDARD_INCLUDE_DIRECTORIES} ${CMAKE_C_STANDARD_INCLUDE_DIRECTORIES}")
endif()

project(bare VERSION 1.0.6)

enable_language(C CXX ASM)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# global include directories
include_directories(
  ${CMAKE_SOURCE_DIR}/platform/inc
  ${CMAKE_SOURCE_DIR}/platform/CMSIS/Device/ST/STM32F0xx/Include
  ${CMAKE_SOURCE_DIR}/platform/CMSIS/Include
  ${CMAKE_SOURCE_DIR}/platform/STM32F0xx_HAL_Driver/Inc
  ${CMAKE_SOURCE_DIR}/app/inc
  ${CMAKE_SOURCE_DIR}/hal/uart/inc
  ${CMAKE_SOURCE_DIR}/hal/inc
  )


set(EXECUTABLE ${PROJECT_NAME})

add_executable(
  ${EXECUTABLE}
  platform/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal.c
  platform/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_cortex.c
  platform/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_gpio.c
  platform/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_rcc.c
  platform/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_uart.c
  platform/STM32F0xx_HAL_Driver/Src/stm32f0xx_hal_uart_ex.c
  platform/startup_stm32f072xb.s
  platform/src/stm32f0xx_hal_msp.c
  platform/src/stm32f0xx_it.c
  platform/src/system_stm32f0xx.c
  app/src/main.cpp
  hal/uart/src/uart_stm32.cpp
  )

target_link_libraries(${EXECUTABLE} PUBLIC)

target_include_directories(${EXECUTABLE} PUBLIC ${PROJECT_BINARY_DIR}
                                                ${CMAKE_SOURCE_DIR})

set_target_properties(${EXECUTABLE} PROPERTIES LINKER_LANGUAGE CXX)

target_link_options(
  ${EXECUTABLE}
  PUBLIC
  -T${CMAKE_SOURCE_DIR}/platform/STM32F072C8Tx_FLASH.ld
  -mcpu=cortex-m0
  -mthumb
  -specs=nano.specs
  -Wl,--no-warn-rwx-segments
  -Wl,-Map=${PROJECT_NAME}.map,--cref
  -Wl,--gc-sections)

# Print executable size
add_custom_command(
  TARGET ${EXECUTABLE}
  POST_BUILD
  COMMAND arm-none-eabi-size $<TARGET_FILE:${EXECUTABLE}>)

# Create hex file
add_custom_command(
  TARGET ${EXECUTABLE}
  POST_BUILD
  COMMAND arm-none-eabi-objcopy -O ihex $<TARGET_FILE:${EXECUTABLE}> ${PROJECT_NAME}.hex
  COMMAND arm-none-eabi-objcopy -O binary $<TARGET_FILE:${EXECUTABLE}> ${PROJECT_NAME}.bin)

# Run elf in Renode
add_custom_target(
  run_in_renode
  COMMAND ${RENODE} --console --disable-xwt ${CMAKE_SOURCE_DIR}/renode_scripts/stm32f072.resc -e start 
  DEPENDS ${PROJECT_NAME}.elf)

