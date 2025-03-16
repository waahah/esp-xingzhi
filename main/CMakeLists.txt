set(SOURCES "audio_codecs/audio_codec.cc"
            "audio_codecs/no_audio_codec.cc"
            "audio_codecs/box_audio_codec.cc"
            "audio_codecs/es8311_audio_codec.cc"
            "audio_codecs/es8388_audio_codec.cc"
            "audio_codecs/cores3_audio_codec.cc"
            "audio_codecs/tcircles3_audio_codec.cc"
            "led/single_led.cc"
            "led/circular_strip.cc"
            "display/display.cc"
            "display/no_display.cc"
            "display/lcd_display.cc"
            "display/ssd1306_display.cc"
            "boards/lilygo-t-circle-s3/esp_lcd_gc9d01n.c"
            "protocols/protocol.cc"
            "iot/thing.cc"
            "iot/thing_manager.cc"
            "system_info.cc"
            "application.cc"
            "ota.cc"
            "settings.cc"
            "background_task.cc"
            "main.cc"
            )

set(INCLUDE_DIRS "." "display" "audio_codecs" "protocols" "audio_processing")

# 添加 IOT 相关文件
file(GLOB IOT_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/iot/things/*.cc)
list(APPEND SOURCES ${IOT_SOURCES})

# 添加板级公共文件
file(GLOB BOARD_COMMON_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/boards/common/*.cc)
list(APPEND SOURCES ${BOARD_COMMON_SOURCES})
list(APPEND INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/boards/common)

# 根据 BOARD_TYPE 配置添加对应的板级文件
if(CONFIG_BOARD_TYPE_XINGZHI_Cube_OLED)
    set(BOARD_TYPE "xingzhi-cube-oled")
elseif(CONFIG_BOARD_TYPE_XINGZHI_Cube_TFT)
    set(BOARD_TYPE "xingzhi-cube-tft")
endif()
file(GLOB BOARD_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/boards/${BOARD_TYPE}/*.cc)
list(APPEND SOURCES ${BOARD_SOURCES})

if(CONFIG_CONNECTION_TYPE_MQTT_UDP)
    list(APPEND SOURCES "protocols/mqtt_protocol.cc")
elseif(CONFIG_CONNECTION_TYPE_WEBSOCKET)
    list(APPEND SOURCES "protocols/websocket_protocol.cc")
endif()

if(CONFIG_USE_AUDIO_PROCESSING)
    list(APPEND SOURCES "audio_processing/audio_processor.cc" "audio_processing/wake_word_detect.cc")
endif()

# 如果目标芯片是 ESP32，则排除特定文件
if(CONFIG_IDF_TARGET_ESP32)
    # 排除 "audio_codecs/box_audio_codec.cc" 和 "audio_codecs/cores3_audio_codec.cc"
    list(REMOVE_ITEM SOURCES "audio_codecs/box_audio_codec.cc"
                             "audio_codecs/cores3_audio_codec.cc"
                             "audio_codecs/es8388_audio_codec.cc")
endif()

file(GLOB ASSETS ${CMAKE_CURRENT_SOURCE_DIR}/assets/zh/*.p3)

idf_component_register(SRCS ${SOURCES}
                    EMBED_FILES ${ASSETS}
                    INCLUDE_DIRS ${INCLUDE_DIRS}
                    WHOLE_ARCHIVE
                    )

# 使用 target_compile_definitions 来定义 BOARD_TYPE
target_compile_definitions(${COMPONENT_LIB}
                    PRIVATE BOARD_TYPE=\"${BOARD_TYPE}\"
                    )
