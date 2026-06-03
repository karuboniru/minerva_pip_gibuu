cmake_minimum_required(VERSION 3.12)
project(minerva_pip_gibuu)
set(CMAKE_CXX_STANDARD 23)

find_package(Boost REQUIRED COMPONENTS program_options)
find_package(ROOT REQUIRED COMPONENTS RIO Tree Physics ROOTDataFrame)

include_directories(${Boost_INCLUDE_DIRS} ${ROOT_INCLUDE_DIRS} include)

add_library(analysis SHARED
  src/EvtTracker2event.cxx
  src/tkievent.cxx
  src/dochi2.cxx
  src/plottools.cxx
)
target_link_libraries(analysis PRIVATE ROOT::Physics ROOT::ROOTDataFrame)

set(GENERAL_LIBS ROOT::Physics ROOT::ROOTDataFrame ROOT::Tree ROOT::Hist ROOT::Gpad
  Boost::program_options
)

add_executable(MINERvA_pi_single_charged_gibuu MINERvA_pi_single_charged_gibuu.cpp)
target_link_libraries(MINERvA_pi_single_charged_gibuu PRIVATE ${GENERAL_LIBS} analysis)

add_executable(gibuu2root gibuu2root.cpp)
target_link_libraries(gibuu2root PRIVATE ROOT::Tree)
target_compile_definitions(gibuu2root PRIVATE
  BASEPATH="${CMAKE_CURRENT_SOURCE_DIR}"
)

add_executable(build_jobcard build_jobcard.cpp)
target_link_libraries(build_jobcard PRIVATE Boost::program_options)
target_compile_definitions(build_jobcard PRIVATE
  BASEPATH="${CMAKE_CURRENT_SOURCE_DIR}"
)
