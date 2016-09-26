ExternalProject_Add(openal-soft
    GIT_REPOSITORY https://github.com/kcat/openal-soft.git
    GIT_TAG openal-soft-1.17.2
    CMAKE_ARGS
        -DALSOFT_NO_CONFIG_UTIL=ON
        -DALSOFT_EXAMPLES=OFF
        -DALSOFT_UTILS=OFF
        -DALSOFT_TESTS=OFF
        -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
)
