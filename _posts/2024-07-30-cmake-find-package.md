


下面举一个例子来说明 CMakeList.txt 之中的 find_package 找包的时候的 .cmake 文件怎么写。
## FindIconv.cmake
这段 CMake 脚本是用来查找 iconv 库和头文件的。主要是任务是找到 编译时的头文件（解决符号问题）和 找到 链接时的库文件 （包括动态库和静态库）
```bash
find_path(ICONV_INCLUDE_DIR NAMES iconv.h)
# 这个 find_path 会在系统下面找目录 如果是自己装的，需要自己指定目录
find_path(ICONV_INCLUDE_DIR NAMES iconv.h PATHS /usr/local/include)

find_library(ICONV_LIBRARY NAMES iconv)
if (ICONV_LIBRARY)
    set(ICONV_SYMBOL_FOUND "${ICONV_LIBRARY}")
else()
    check_function_exists(iconv_open ICONV_SYMBOL_FOUND)
endif()

include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(Iconv DEFAULT_MESSAGE ICONV_INCLUDE_DIR ICONV_SYMBOL_FOUND)

if(ICONV_LIBRARY)
    set(ICONV_LIBRARIES "${ICONV_LIBRARY}")
else()
    set(ICONV_LIBRARIES)
endif()
set(ICONV_INCLUDE_DIRS "${ICONV_INCLUDE_DIR}")

mark_as_advanced(ICONV_LIBRARY ICONV_INCLUDE_DIR)
```