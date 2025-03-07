cmake_minimum_required(VERSION 3.15)

project(OB-X-AE VERSION 0.0.2)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(external/JUCE)
add_subdirectory(external/clap-juce-extensions EXCLUDE_FROM_ALL)

set(BaseTargetName OB-X-AE)

juce_add_plugin(${BaseTargetName}
        ICON_BIG external/OB-Xx/Source/Images/appicon.png
        ICON_SMALL external/OB-Xx/Source/Images/appicon.png
        COMPANY_NAME "Atsushi Eno"
        BUNDLE_ID "atsushieno"
        IS_SYNTH TRUE
        NEEDS_MIDI_INPUT TRUE
        NEEDS_MIDI_OUTPUT FALSE
        IS_MIDI_EFFECT FALSE
        EDITOR_WANTS_KEYBOARD_FOCUS TRUE
        COPY_PLUGIN_AFTER_BUILD TRUE
        PLUGIN_MANUFACTURER_CODE AENO
        PLUGIN_CODE OBAE
        FORMATS AU AUv3 VST3 LV2 Standalone
        LV2URI https://github.com/atsushieno/OB-X-AE
        PRODUCT_NAME OB-X-AE
        )

clap_juce_extensions_plugin(TARGET ${BaseTargetName}
        CLAP_ID https://github.com/atsushieno/OB-X-AE
        CLAP_FEATURES instrument
        )

juce_generate_juce_header(${BaseTargetName})

target_compile_definitions(${BaseTargetName}
        PUBLIC
        JUCE_WEB_BROWSER=0
        JUCE_USE_CURL=0
        JUCE_VST3_CAN_REPLACE_VST2=0
        )

target_link_libraries(${BaseTargetName} PRIVATE
        juce_audio_utils
        juce_recommended_config_flags
        juce_recommended_lto_flags
        juce_recommended_warning_flags
        )

target_sources(${BaseTargetName}
        PRIVATE
        external/OB-Xx/Source/Components/PresetBar.cpp
        external/OB-Xx/Source/Components/ScaleComponent.cpp
        external/OB-Xx/Source/Components/SetPresetNameWindow.cpp
        external/OB-Xx/Source/MTS/libMTSClient.cpp
        external/OB-Xx/Source/PluginEditor.cpp
        external/OB-Xx/Source/PluginProcessor.cpp
        external/OB-Xx/Source/Utils.cpp
        )

# might be unnecessary
juce_add_binary_data(${BaseTargetName}_BinaryData
        SOURCES
        external/OB-Xx/Source/Images/appicon.png
        external/OB-Xx/Source/Images/main.png
        external/OB-Xx/Source/Images/main.psd
        external/OB-Xx/Source/Images/main@2x.png
        external/OB-Xx/Source/Images/main@4x.png
        external/OB-Xx/Source/Images/menu.png
        external/OB-Xx/Source/Images/menu@2x.png
        external/OB-Xx/Source/Images/menu@4x.png
        external/OB-Xx/Source/Images/presetnavigation.svg
)

target_include_directories(${BaseTargetName}
        PRIVATE
        external/OB-Xx/Source
        external/OB-Xx
        # there was some weird path dependency...
        )

target_link_libraries(${BaseTargetName}
        PRIVATE
        ${BaseTargetName}_BinaryData
)

# Determine documents directory based on platform
if(WIN32)
        execute_process(
                COMMAND powershell -command "[Environment]::GetFolderPath('MyDocuments')"
                OUTPUT_VARIABLE DOCUMENTS_DIR
                OUTPUT_STRIP_TRAILING_WHITESPACE
        )
elseif(APPLE)
        execute_process(
                COMMAND sh -c "echo $HOME/Documents"
                OUTPUT_VARIABLE DOCUMENTS_DIR
                OUTPUT_STRIP_TRAILING_WHITESPACE
        )
else() # Linux/Unix
        execute_process(
                COMMAND sh -c "xdg-user-dir DOCUMENTS 2>/dev/null || echo $HOME/Documents"
                OUTPUT_VARIABLE DOCUMENTS_DIR
                OUTPUT_STRIP_TRAILING_WHITESPACE
        )
endif()

file(COPY
        external/OB-Xx/Resources/
        DESTINATION ${DOCUMENTS_DIR}/OB-X-AE
)

function(apply_patch_if_needed PATCH_FILE SOURCE_DIR)
        # Get the absolute path to the patch file
        get_filename_component(PATCH_FILE_ABS ${PATCH_FILE} ABSOLUTE)

        # Create a stamp file path to track if the patch was applied
        get_filename_component(PATCH_FILENAME ${PATCH_FILE} NAME)
        set(STAMP_FILE "${SOURCE_DIR}/.patch_${PATCH_FILENAME}.stamp")

        # Check if the stamp file exists and is newer than the patch file
        if(EXISTS "${STAMP_FILE}" AND "${STAMP_FILE}" IS_NEWER_THAN "${PATCH_FILE_ABS}")
                message(STATUS "Patch ${PATCH_FILENAME} already applied to ${SOURCE_DIR}")
        else()
                message(STATUS "Applying patch ${PATCH_FILENAME} to ${SOURCE_DIR}")

                # Apply the patch
                execute_process(
                        COMMAND ${CMAKE_COMMAND} -E chdir ${CMAKE_CURRENT_LIST_DIR}/${SOURCE_DIR} git apply ${PATCH_FILE_ABS}
                )
        endif()
endfunction()

apply_patch_if_needed(ob-x-ae.patch external/OB-Xx)
