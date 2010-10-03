# Copyright (C) 2007-2009 LuaDist.
# Created by Peter Kapec
# Redistribution and use of this file is allowed according to the terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own license.

PROJECT(wxlua)
CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
INCLUDE(dist.cmake)

find_package(Lua51 REQUIRED)

# Find WxWidgets
FIND_PACKAGE(wxWidgets REQUIRED base core gl html  media net richtext stc xml xrc adv aui)
INCLUDE(${wxWidgets_USE_FILE})

# Source files
INCLUDE_DIRECTORIES( ${CMAKE_CURRENT_SOURCE_DIR} "modules" "modules/wxbind/setup" )

# WxLua library
FILE(GLOB_RECURSE SRC_WXBIND "modules/wxbind/*.cpp")
FILE(GLOB_RECURSE SRC_WXLUA "modules/wxlua/*.cpp")
FILE(GLOB_RECURSE SRC_WXLUADEBUG "modules/wxluadebug/*.cpp")
FILE(GLOB_RECURSE SRC_WXLUASOCKET "modules/wxluasocket/*.cpp" )

SET(WXLUA_SRC  ${SRC_LUAMODULE} ${SRC_WXBIND} ${SRC_WXLUA} ${SRC_WXLUADEBUG} ${SRC_WXLUASOCKET})

LIST(REMOVE_ITEM WXLUA_SRC
	${CMAKE_CURRENT_SOURCE_DIR}/modules/wxbind/src/dummy.cpp 
	${CMAKE_CURRENT_SOURCE_DIR}/modules/wxlua/src/dummy.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/modules/wxluadebug/src/dummy.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/modules/wxluasocket/src/dummy.cpp	)

ADD_LIBRARY(wxlualib SHARED ${WXLUA_SRC})
TARGET_LINK_LIBRARIES(wxlualib ${LUA_LIBRARY} ${wxWidgets_LIBRARIES})
SET_TARGET_PROPERTIES( wxlualib PROPERTIES OUTPUT_NAME wxlua CLEAN_DIRECT_OUTPUT 1 )

# WxLua module
FILE(GLOB_RECURSE SRC_LUAMODULE "modules/luamodule/*.cpp")
ADD_LUA_MODULE(wx ${SRC_LUAMODULE})
TARGET_LINK_LIBRARIES(wx wxlualib)
#~2DO: wx .def and .rc files

# WxLua application
INCLUDE_DIRECTORIES( "apps/wxlua/src" )
FILE(GLOB SRC_WXLUAPP "apps/wxlua/src/*.cpp")

ADD_EXECUTABLE(wxlua ${SRC_WXLUAPP})
TARGET_LINK_LIBRARIES(wxlua wxlualib )

# WxLuaCan application
INCLUDE_DIRECTORIES( "apps/wxluacan/src" )
FILE(GLOB SRC_WXLUACANAPP "apps/wxluacan/src/*.cpp")

ADD_EXECUTABLE(wxluacan ${SRC_WXLUACANAPP})
TARGET_LINK_LIBRARIES(wxluacan wxlualib )

# WxLuaFreeze application
ADD_EXECUTABLE(wxluafreeze apps/wxluafreeze/src/wxluafreeze.cpp)
TARGET_LINK_LIBRARIES(wxluafreeze wxlualib)

# Find Doxygen and generate documentation
FIND_PACKAGE(Doxygen)
IF(DOXYGEN_FOUND)
	ADD_CUSTOM_TARGET(	DoxygenDoc ALL
		COMMAND ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/docs/doxygen.cfg
		WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
		VERBATIM		
	)
	INSTALL (DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/doxygen DESTINATION ${INSTALL_DOC})
ENDIF(DOXYGEN_FOUND)

# Install all files and documentation
INSTALL (TARGETS wxlualib DESTINATION ${INSTALL_LIB} )
INSTALL (TARGETS wx DESTINATION ${INSTALL_CMOD})
INSTALL (TARGETS wxlua wxluacan wxluafreeze DESTINATION ${INSTALL_BIN})
INSTALL (DIRECTORY samples/ DESTINATION ${INSTALL_EXAMPLE})
INSTALL (FILES apps/wxluafreeze/src/wxluafreeze.lua util/bin2c/bin2c.lua util/wrapmodule/wrapmodule.wx.lua DESTINATION ${INSTALL_FOO})
INSTALL (DIRECTORY docs/ DESTINATION ${INSTALL_DOC} PATTERN "doxygen.cfg" EXCLUDE  PATTERN "doxygen/*" EXCLUDE)
