cmake_minimum_required(VERSION 3.22)
project(SpatialExpander VERSION 1.0.0)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(JUCE_DIR /Users/keybo/opt/JUCE)
add_subdirectory(${JUCE_DIR} juce)

juce_add_plugin(SpatialExpander
    VERSION_STRING "1.0.0"
    COMPANY_NAME "EaryChow"
    PRODUCT_NAME "SpatialExpander"
    FORMATS VST3
)

target_sources(SpatialExpander PRIVATE
    Source/PluginProcessor.cpp
    Source/PluginEditor.cpp
    Source/DSP/SurroundSTFT.cpp
    Source/DSP/LFEExtractor.cpp
    Source/DSP/SpatialAnalyser.cpp
    Source/DSP/LoudnessMeter.cpp
)

target_link_libraries(SpatialExpander PRIVATE
    juce::juce_dsp
    juce::juce_audio_devices
    juce::juce_audio_formats
    juce::juce_audio_utils
)

# 与作者 .jucer 中 JUCE_VST3_CAN_REPLACE_VST2="0" 保持一致，避免 VST3 参数自动化静态断言
target_compile_definitions(SpatialExpander PRIVATE
    JUCE_VST3_CAN_REPLACE_VST2=0
)
