# ROS wrapper CMake project for these MRPT libraries:
# mrpt-nav, mrpt-kinematics

cmake_minimum_required(VERSION 3.5)
project(python_mrpt)

include(ExternalProject)

#----
# Extract version from package.xml
# Example line:" <version>0.3.2</version>"
file(READ package.xml contentPackageXML)
string(REGEX MATCH "<version>([0-9\.]*)</version>" _ ${contentPackageXML})
set(MRPT_VERSION_TO_DOWNLOAD ${CMAKE_MATCH_1})
message(STATUS "MRPT_VERSION_TO_DOWNLOAD: ${MRPT_VERSION_TO_DOWNLOAD} (detected in package.xml)")
#----

# Reduce chances of RAM exaustion:
set(PYMRPT_MAXIMUM_PARALLELIZATION 2)

ExternalProject_Add(
  mrpt
  PREFIX ${CMAKE_BINARY_DIR}/mrpt-build
  GIT_REPOSITORY https://github.com/MRPT/mrpt.git
  GIT_TAG ${MRPT_VERSION_TO_DOWNLOAD}
  CMAKE_ARGS 
    # Main settings:
    -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
    # Disabled stuff:
    -DDISABLE_JSONCPP=ON
    -DMRPT_INSTALL_EXAMPLES=OFF
    -DMRPT_INSTALL_SHARE_DIRS=OFF
    -DMRPT_BUILD_MANPAGES=OFF
    -DMRPT_INSTALL_CONFIG_HEADERS=OFF
    -DDISABLE_ROS=ON # save cmake configure time
    # Granular builds for each ROS package:
    -DMRPT_BUILD_APPLICATIONS=OFF
    -DMRPT_DISABLE_PYTHON_BINDINGS=OFF   # Build pymrpt!
    -DMRPT_WITH_ROBOPEAK_LIDAR=OFF
    -DMRPT_BUILD_wxThings=OFF
    -DMRPT_BUILD_rplidar_sdk=OFF
    -DMRPT_BUILD_xsens=OFF
    -DMRPT_BUILD_nanogui=OFF
    # Specific for pymrpt to prevent memory exhaustion:
    -DMRPT_PYTHON_LTO_PARALLEL_JOBS=${PYMRPT_MAXIMUM_PARALLELIZATION}  # Minimize RAM usage, at the cost of longer build time
    -DCMAKE_BUILD_PARALLEL_LEVEL=${PYMRPT_MAXIMUM_PARALLELIZATION}
    # Libs:
    -DBUILD_mrpt-apps=OFF
    -DBUILD_mrpt-bayes=OFF
    -DBUILD_mrpt-comms=OFF
    -DBUILD_mrpt-config=OFF
    -DBUILD_mrpt-containers=OFF
    -DBUILD_mrpt-core=OFF
    -DBUILD_mrpt-expr=OFF
    -DBUILD_mrpt-graphs=OFF
    -DBUILD_mrpt-graphslam=OFF
    -DBUILD_mrpt-gui=OFF
    -DBUILD_mrpt-hwdrivers=OFF
    -DBUILD_mrpt-img=OFF
    -DBUILD_mrpt-io=OFF
    -DBUILD_mrpt-kinematics=OFF
    -DBUILD_mrpt-maps=OFF
    -DBUILD_mrpt-math=OFF
    -DBUILD_mrpt-nav=OFF
    -DBUILD_mrpt-obs=OFF
    -DBUILD_mrpt-opengl=OFF
    -DBUILD_mrpt-poses=OFF
    -DBUILD_mrpt-random=OFF
#    -DBUILD_mrpt-ros2bridge=OFF
    -DBUILD_mrpt-rtti=OFF
    -DBUILD_mrpt-serialization=OFF
    -DBUILD_mrpt-slam=OFF
    -DBUILD_mrpt-system=OFF
    -DBUILD_mrpt-tclap=OFF
    -DBUILD_mrpt-tfest=OFF
    -DBUILD_mrpt-topography=OFF
    -DBUILD_mrpt-typemeta=OFF
    -DBUILD_mrpt-vision=OFF
  # no install during build
  INSTALL_COMMAND ""
  # Custom BUILD command to limit parallelization:
  BUILD_COMMAND ${CMAKE_COMMAND} --build ${CMAKE_BINARY_DIR}/mrpt-build/src/mrpt-build -j ${PYMRPT_MAXIMUM_PARALLELIZATION}
)

# Actual "install":
install(CODE "execute_process(COMMAND ${CMAKE_COMMAND} --build ${CMAKE_BINARY_DIR}/mrpt-build/src/mrpt-build --target install)")
