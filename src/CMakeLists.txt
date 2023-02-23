set(DTK_QML_APP_PLUGIN_PATH "${LIB_INSTALL_DIR}/${LIB_NAME}/qml-app" CACHE STRING "dtk qml app plugin path")
set(DTK_QML_APP_PLUGIN_SUBPATH "dtkdeclarative/plugins" CACHE STRING "dtk qml app plugin subpath")
set(DDECLARATIVE_TRANSLATIONS_DIR "dtk5/DDeclarative/translations" CACHE STRING "DDeclarative translations directory")
set(DDECLARATIVE_TRANSLATIONS_PATH "share/${DDECLARATIVE_TRANSLATIONS_DIR}")
set(TRANSLATIONS_INSTALL_PATH "${DDECLARATIVE_TRANSLATIONS_PATH}")

# Add secure compiler options
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fstack-protector-all")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fstack-protector-all")
set(CMAKE_EXE_LINKER_FLAGS  "-z relro -z now -z noexecstack -pie")
# For mips64
if (${CMAKE_SYSTEM_PROCESSOR} MATCHES "mips64")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -ftree-vectorize -march=loongson3a -mhard-float -mno-micromips -mno-mips16 -flax-vector-conversions -mloongson-ext2 -mloongson-mmi")
endif()

find_package(Qt5 REQUIRED COMPONENTS Core Quick QuickCompiler DBus QuickControls2 LinguistTools)
find_package(Dtk REQUIRED COMPONENTS Core Gui)

file(GLOB PUBLIC_HEADERS "./*.h")
file(GLOB_RECURSE HEADERS "./*.h")
file(GLOB_RECURSE SRCS   "./*.cpp")
file(GLOB D_HEADERS   "./D*")
file(GLOB QML_RCS ${CMAKE_CURRENT_LIST_DIR}/dtkdeclarative_qml.qrc)
file(GLOB ASSETS_RCS ${CMAKE_CURRENT_LIST_DIR}/dtkdeclarative_assets.qrc)
file(GLOB TS_FILES "translations/*.ts")
# Quick compiler gen qmlc
qtquick_compiler_add_resources(RESOURCES ${QML_RCS})
# qtquick_compiler_add_resources(ASSETS_RESOURCES ${ASSETS_RCS})
# Add translation source file
qt5_add_translation(QM_FILES ${TS_FILES})

add_library(${LIB_NAME} SHARED
    ${SRCS}
    ${HEADERS}
    ${D_HEADERS}
    ${QML_RCS}
    ${ASSETS_RCS}
    ${QM_FILES}
)

set_target_properties(${LIB_NAME} PROPERTIES
    VERSION ${CMAKE_PROJECT_VERSION}
    SOVERSION ${CMAKE_PROJECT_VERSION_MAJOR}
    EXPORT_NAME Declarative
)

target_compile_definitions(${LIB_NAME} PRIVATE
    DTK_QML_APP_PLUGIN_PATH="${DTK_QML_APP_PLUGIN_PATH}"
    DTK_QML_APP_PLUGIN_SUBPATH="${DTK_QML_APP_PLUGIN_SUBPATH}"
    DDECLARATIVE_TRANSLATIONS_DIR="${DDECLARATIVE_TRANSLATIONS_DIR}"
)

if(USE_QQuickStylePluginPrivate)
    target_link_libraries(${LIB_NAME} PRIVATE
        Qt5::QuickControls2Private
    )
endif()

target_link_libraries(${LIB_NAME}
PUBLIC
    Qt5::Core
    Qt5::Quick
    Dtk::Core
    Dtk::Gui
PRIVATE
    Qt5::QuickPrivate
    Qt5::DBus
)

target_include_directories(${LIB_NAME} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}>
)

target_include_directories(${LIB_NAME} INTERFACE
    $<INSTALL_INTERFACE:${INCLUDE_INSTALL_DIR}>
)

target_link_directories(${LIB_NAME} INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
    $<INSTALL_INTERFACE:${LIB_INSTALL_DIR}>
)

# gen dtkdeclarative_config.h
include(DtkBuildConfig)
set(options MSG) # print MODULE_NAME D_HEADERS DEST_DIR
gen_dtk_config_header(MODULE_NAME ${LIB_NAME} HEADERS ${D_HEADERS})

# Install library
install(TARGETS ${LIB_NAME} EXPORT DtkDeclarativeTargets DESTINATION "${LIB_INSTALL_DIR}")
# Install headers
install(FILES ${PUBLIC_HEADERS} ${D_HEADERS} DESTINATION "${INCLUDE_INSTALL_DIR}")
# Install export targets
install(EXPORT DtkDeclarativeTargets NAMESPACE Dtk:: FILE DtkDeclarativeTargets.cmake DESTINATION "${CONFIG_INSTALL_DIR}")
# Install translations
install(FILES ${QM_FILES} DESTINATION "${TRANSLATIONS_INSTALL_PATH}")