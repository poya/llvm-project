set(LLVM_LINK_COMPONENTS
  AllTargetsAsmParsers
  AllTargetsCodeGens
  AllTargetsDescs
  AllTargetsInfos
  AsmPrinter
  DebugInfoDWARF
  MC
  Object
  Support
  Target
  )

add_llvm_tool(llvm-dwp
  llvm-dwp.cpp
  DWP.cpp
  DWPError.cpp

  DEPENDS
  intrinsics_gen
  )

if(LLVM_INSTALL_BINUTILS_SYMLINKS)
  add_llvm_tool_symlink(dwp llvm-dwp)
endif()
