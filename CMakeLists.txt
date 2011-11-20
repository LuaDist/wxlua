# Copyright (C) 2007-2011 LuaDist.
# Created by Peter Kapec. Peter Drahos
# Redistribution and use of this file is allowed according to the terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own license.

PROJECT(wxlua C CXX)
CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
INCLUDE(dist.cmake)

FIND_PACKAGE(Lua51 REQUIRED)
FIND_PACKAGE(wxWidgets COMPONENTS base core gl html media net richtext stc xml xrc adv aui)

# Find wx on mingw, this assumes msw built monolithic libraries + gl and stc
IF (MINGW AND NOT wxWidgets_FOUND)
	FIND_PATH(wxWidgets_INCLUDE_DIR NAMES wx/wx.h wx.h)
	FIND_LIBRARY(wxWidgets_LIBRARY NAMES wxmsw28)
	FIND_LIBRARY(wxWidgets_gl_LIBRARY NAMES wxmsw28_gl)
	FIND_LIBRARY(wxWidgets_stc_LIBRARY NAMES wxmsw28_stc)
	SET (wxWidgets_LIBRARIES
		${wxWidgets_LIBRARY}
		${wxWidgets_gl_LIBRARY} 
		${wxWidgets_stc_LIBRARY} 
		)
	SET ( CMAKE_CXX_FLAGS "-fpermissive" )
ENDIF()
INCLUDE_DIRECTORIES( ${wxWidgets_INCLUDE_DIR} ${CMAKE_CURRENT_SOURCE_DIR} "modules" "modules/wxbind/setup" )

# Collect sources
FILE ( GLOB WXLUA_LIB_SRC modules/wxlua/src/wx*.cpp )
FILE ( GLOB WXLUA_DEBUG_SRC modules/wxluadebug/src/wx*.cpp )
FILE ( GLOB WXLUA_BIND_SRC modules/wxbind/src/wx*.cpp )
FILE ( GLOB WXLUA_SOCKET_SRC modules/wxluasocket/src/wx*.cpp )

# Additional link libraries
IF (WIN32)
	ADD_DEFINITIONS(-D__WXMSW__)
	SET(LIB_SOCKET ws2_32)
	SET ( EXE_TYPE WIN32 )
ENDIF (WIN32)

# wxLua shared Library
SET( WXLUA_ALL_SRC ${WXLUA_LIB_SRC}  ${WXLUA_DEBUG_SRC} ${WXLUA_BIND_SRC} ${WXLUA_SOCKET_SRC}  )
ADD_LIBRARY ( libwxlua SHARED ${WXLUA_ALL_SRC})
TARGET_LINK_LIBRARIES( libwxlua ${wxWidgets_LIBRARIES} ${LUA_LIBRARY} ${LIB_SOCKET} )
SET_TARGET_PROPERTIES( libwxlua PROPERTIES OUTPUT_NAME "wxlua" )

# wxLua application
FILE ( GLOB WXLUA_SRC apps/wxlua/src/*.cpp )
ADD_EXECUTABLE( wxlua ${EXE_TYPE} ${WXLUA_SRC} apps/wxlua/src/wxlua.rc)
TARGET_LINK_LIBRARIES( wxlua libwxlua )

# wxLuaCan application
#FILE ( GLOB WXLUACAN_SRC apps/wxluacan/src/*.cpp )
#ADD_EXECUTABLE( wxluacan ${EXE_TYPE} ${WXLUACAN_SRC} apps/wxluacan/src/cansim.rc)
#TARGET_LINK_LIBRARIES( wxluacan libwxlua )

# wxLuaEdit application (this needs wx/stedit)
#FILE ( GLOB WXLUAEDIT_SRC apps/wxluaedit/src/*.cpp )
#ADD_EXECUTABLE( wxluaedit ${EXE_TYPE} ${WXLUAEDIT_SRC} apps/wxluaedit/src/wxluaedit.rc)
#TARGET_LINK_LIBRARIES( wxluaedit libwxlua )

# wx module

INSTALL_LUA_MODULE( wx modules/luamodule/src/luamodule.cpp LINK libwxlua )

# Install all files and documentation
INSTALL (TARGETS libwxlua wxlua RUNTIME DESTINATION ${INSTALL_BIN} LIBRARY DESTINATION ${INSTALL_LIB} ARCHIVE DESTINATION ${INSTALL_LIB})
INSTALL (DIRECTORY samples/ DESTINATION ${INSTALL_EXAMPLE})
#INSTALL (FILES apps/wxluafreeze/src/wxluafreeze.lua util/bin2c/bin2c.lua util/wrapmodule/wrapmodule.wx.lua DESTINATION ${INSTALL_FOO})
#INSTALL (DIRECTORY docs/ DESTINATION ${INSTALL_DOC} PATTERN "doxygen.cfg" EXCLUDE  PATTERN "doxygen/*" EXCLUDE)
