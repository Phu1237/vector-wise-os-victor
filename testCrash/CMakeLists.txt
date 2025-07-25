cmake_minimum_required(VERSION 3.10)

project(victor_testcrash)

include(anki_build_cxx)

set(PLATFORM_LIBS "")
set(PLATFORM_COMPILE_DEFS "")

if (VICOS)
    include(breakpad)
    set(PLATFORM_LIBS
    )
elseif (MACOSX)
    include(webots)
    set(PLATFORM_LIBS
        ${WEBOTS_LIBS}
    )
    set(PLATFORM_COMPILE_DEFS "-DSIMULATOR")
endif()

if (VICOS AND ANKI_DEV_CHEATS)

  anki_build_cxx_executable(vic-testcrash ${ANKI_SRCLIST_DIR})
  anki_build_target_license(vic-testcrash "ANKI")

  target_link_libraries(vic-testcrash
    PRIVATE
    cti_common
    victorCrashReports
    ${BREAKPAD_LIBS}
    ${PLATFORM_LIBS}
    PUBLIC
    robot_interface  # Needs to be public for cozmoConfig.h
  )

  anki_build_strip(TARGET vic-testcrash)
    
endif()
