
option(RDK_BUILD_QT_SUPPORT "build support for QT drawing" OFF )
option(RDK_BUILD_QT_DEMO "build the QT drawing demo" OFF )
option(RDK_BUILD_CAIRO_SUPPORT "build support for Cairo drawing" OFF )
option(RDK_BUILD_FREETYPE_SUPPORT "build support for FreeType font handling" ON)

rdkit_headers(MolDraw2D.h
  MolDraw2DSVG.h
  MolDraw2Dwx.h
  MolDraw2DUtils.h
  DrawText.h
  DrawTextSVG.h
  DEST GraphMol/MolDraw2D
)

remove_definitions(-DRDKIT_GRAPHMOL_BUILD)
add_definitions(-DRDKIT_MOLDRAW2D_BUILD)
rdkit_library(MolDraw2D MolDraw2D.cpp MolDraw2DSVG.cpp
  MolDraw2DDetails.cpp MolDraw2DUtils.cpp DrawText.cpp
  DrawTextSVG.cpp
  LINK_LIBRARIES
  ChemReactions FileParsers SmilesParse Depictor MolTransforms 
  SubstructMatch Subgraphs GraphMol EigenSolvers )

if(RDK_BUILD_QT_SUPPORT)
  find_package(Qt5 COMPONENTS Widgets OpenGL REQUIRED)
  target_sources(MolDraw2D PRIVATE MolDraw2DQt.cpp)
  target_compile_options(MolDraw2D PRIVATE "-std=c++11")
  target_link_libraries(MolDraw2D PUBLIC Qt5::Widgets Qt5::OpenGL)
  rdkit_headers(MolDraw2DQt.h DEST GraphMol/MolDraw2D)
endif(RDK_BUILD_QT_SUPPORT)

if(RDK_BUILD_CAIRO_SUPPORT)
  find_package(Cairo REQUIRED)
  target_compile_definitions(MolDraw2D PUBLIC "-DRDK_BUILD_CAIRO_SUPPORT")
  target_link_libraries(MolDraw2D PUBLIC Cairo::Cairo)
  target_sources(MolDraw2D PRIVATE MolDraw2DCairo.cpp DrawTextCairo.cpp)
  if(RDK_INSTALL_STATIC_LIBS)
     target_compile_definitions(MolDraw2D_static PUBLIC "-DRDK_BUILD_CAIRO_SUPPORT")
     target_link_libraries(MolDraw2D_static PUBLIC Cairo::Cairo)
     target_sources(MolDraw2D_static PRIVATE MolDraw2DCairo.cpp DrawTextCairo.cpp)
  endif()
  rdkit_headers(MolDraw2DCairo.h DrawTextCairo.h DEST GraphMol/MolDraw2D)
endif(RDK_BUILD_CAIRO_SUPPORT)

if(RDK_BUILD_FREETYPE_SUPPORT)
  target_compile_definitions(MolDraw2D PUBLIC "-DRDK_BUILD_FREETYPE_SUPPORT")
  target_sources(MolDraw2D PRIVATE DrawText.cpp
          DrawTextFT.cpp DrawTextFTSVG.cpp)
  if(RDK_INSTALL_STATIC_LIBS)
    target_compile_definitions(MolDraw2D_static PUBLIC "-DRDK_BUILD_FREETYPE_SUPPORT")
    target_sources(MolDraw2D_static PRIVATE DrawText.cpp
          DrawTextFT.cpp DrawTextFTSVG.cpp)
  endif()
  if( ${CMAKE_SYSTEM_NAME} MATCHES "Emscripten")
      set(USE_FLAGS "-s USE_FREETYPE=1")
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${USE_FLAGS}")
      set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${USE_FLAGS}")
      target_link_libraries(MolDraw2D PUBLIC ${FREETYPE_LIBRARIES})
      if(RDK_INSTALL_STATIC_LIBS)
        target_link_libraries(MolDraw2D_static PUBLIC ${FREETYPE_LIBRARIES})
      endif()
  else()
      find_package(Freetype REQUIRED)
      target_link_libraries(MolDraw2D PUBLIC Freetype::Freetype)
      if(RDK_INSTALL_STATIC_LIBS)
        target_link_libraries(MolDraw2D_static PUBLIC Freetype::Freetype)
      endif()
  endif()
  rdkit_headers(DrawText.h DrawTextFT.h
          DrawTextFTSVG.h DEST GraphMol/MolDraw2D)
  if(RDK_BUILD_CAIRO_SUPPORT)
    target_sources(MolDraw2D PRIVATE DrawTextFTCairo.cpp)
    if(RDK_INSTALL_STATIC_LIBS)
      target_sources(MolDraw2D_static PRIVATE DrawTextFTCairo.cpp)
    endif()
    rdkit_headers(DrawTextFTCairo.h DEST GraphMol/MolDraw2D)
  endif()
endif(RDK_BUILD_FREETYPE_SUPPORT)

rdkit_test(moldraw2DTest1 test1.cpp LINK_LIBRARIES
  MolDraw2D )

rdkit_catch_test(moldraw2DTestCatch catch_tests.cpp LINK_LIBRARIES
  MolDraw2D )

rdkit_test(moldraw2DRxnTest1 rxn_test1.cpp LINK_LIBRARIES
  MolDraw2D )

if(RDK_BUILD_QT_DEMO)
  add_subdirectory(QTDemo)
endif(RDK_BUILD_QT_DEMO)

if(RDK_BUILD_PYTHON_WRAPPERS)
add_subdirectory(Wrap)
endif()
