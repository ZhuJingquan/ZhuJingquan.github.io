---
layout:     post
title:      "Notes-on-python-os-module"
subtitle:   " \" just follow the standard way\""
date:       2018-02-13 08:51:22
author:     "朱景泉"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Notes
typora-copy-images-to: ../img
---

> “Yeah It's on. ”

# [python中OS常用方法](http://www.cnblogs.com/mufenglin/p/7676160.html)

Python的标准库中的os模块包含普遍的操作系统功能。如果你希望你的程序能够与平台无关的话，这个模块是尤为重要的。即它允许一个程序在编写后不需要任何改动，也不会发生任何问题，就可以在Linux和Windows下运行。

下面列出了一些在os模块中比较有用的部分。它们中的大多数都简单明了。

os.name字符串指示你正在使用的平台。比如对于Windows，它是'nt'，而对于Linux/Unix用户，它是'posix'。

os.getcwd()函数得到当前工作目录，即当前Python脚本工作的目录路径。

os.getenv()获取一个环境变量，如果没有返回none

os.putenv(key, value)设置一个环境变量值

os.listdir(path)返回指定目录下的所有文件和目录名。

os.remove(path)函数用来删除一个文件。

os.system(command)函数用来运行shell命令。

os.linesep字符串给出当前平台使用的行终止符。例如，Windows使用'\r\n'，Linux使用'\n'而Mac使用'\r'。

os.curdir:返回当前目录（'.')
os.chdir(dirname):改变工作目录到dirname

========================================================================================

os.path常用方法：

os.path.isfile()和os.path.isdir()函数分别检验给出的路径是一个文件还是目录。

os.path.existe()函数用来检验给出的路径是否真地存在

os.path.getsize(name):获得文件大小，如果name是目录返回0L

os.path.abspath(name):获得绝对路径
os.path.normpath(path):规范path字符串形式

os.path.split(path) ：将path分割成目录和文件名二元组返回。

os.path.splitext():分离文件名与扩展名

*os.path.join(path,name):连接目录与文件名或目录os.path.basename(path):返回文件名os.path.dirname(path):返回文件路径*

DESCRIPTION

    Instead of importing this module directly, import os and refer to
    this module as os.path.  The "os.path" name is an alias for this
    module on Posix systems; on other systems (e.g. Mac, Windows),
    os.path provides the same operations in a manner specific to that
    platform, and is an alias to another module (e.g. macpath, ntpath).
    
    Some of this can actually be useful on non-Posix systems too, e.g.
    for manipulation of the pathname component of URLs.

FUNCTIONS
    abspath(path)
        Return an absolute path.
    
    basename(p)
        Returns the final component of a pathname
    
    commonprefix(m)
        Given a list of pathnames, returns the longest common leading component
    
    dirname(p)
        Returns the directory component of a pathname
    
    exists(path)
        Test whether a path exists.  Returns False for broken symbolic links
    
    expanduser(path)
        Expand ~ and ~user constructions.  If user or $HOME is unknown,
        do nothing.
    
    expandvars(path)
        Expand shell variables of form $var and ${var}.  Unknown variables
        are left unchanged.
    
    getatime(filename)
        Return the last access time of a file, reported by os.stat().
    
    getctime(filename)
        Return the metadata change time of a file, reported by os.stat().
    
    getmtime(filename)
        Return the last modification time of a file, reported by os.stat().
    
    getsize(filename)
        Return the size of a file, reported by os.stat().
    
    isabs(s)
        Test whether a path is absolute
    
    isdir(s)
        Return true if the pathname refers to an existing directory.
    
    isfile(path)
        Test whether a path is a regular file
    
    islink(path)
        Test whether a path is a symbolic link
    
    ismount(path)
        Test whether a path is a mount point
    
    join(a, *p)
        Join two or more pathname components, inserting '/' as needed.
        If any component is an absolute path, all previous path components
        will be discarded.  An empty last part will result in a path that
        ends with a separator.
    
    lexists(path)
        Test whether a path exists.  Returns True for broken symbolic links
    
    normcase(s)
        Normalize case of pathname.  Has no effect under Posix
    
    normpath(path)
        Normalize path, eliminating double slashes, etc.
    
    realpath(filename)
        Return the canonical path of the specified filename, eliminating any
        symbolic links encountered in the path.
    
    relpath(path, start='.')
        Return a relative version of a path
    
    samefile(f1, f2)
        Test whether two pathnames reference the same actual file
    
    sameopenfile(fp1, fp2)
        Test whether two open file objects reference the same file
    
    samestat(s1, s2)
        Test whether two stat buffers reference the same file
    
    split(p)
        Split a pathname.  Returns tuple "(head, tail)" where "tail" is
        everything after the final slash.  Either part may be empty.
    
    splitdrive(p)
        Split a pathname into drive and path. On Posix, drive is always
        empty.
    
    splitext(p)
        Split the extension from a pathname.
        
        Extension is everything from the last dot to the end, ignoring
        leading dots.  Returns "(root, ext)"; ext may be empty.
    
    walk(top, func, arg)
        Directory tree walk with callback function.
        
        For each directory in the directory tree rooted at top (including top
        itself, but excluding '.' and '..'), call func(arg, dirname, fnames).
        dirname is the name of the directory, and fnames a list of the names of
        the files and subdirectories in dirname (excluding '.' and '..').  func
        may modify the fnames list in-place (e.g. via del or slice assignment),
        and walk will only recurse into the subdirectories whose names remain in
        fnames; this can be used to implement a filter, or to impose a specific
        order of visiting.  No semantics are defined for, or required of, arg,
        beyond that arg is always passed to func.  It can be used, e.g., to pass
        a filename pattern, or a mutable object designed to accumulate
        statistics.  Passing None for arg is common.

DATA
    __all__ = ['normcase', 'isabs', 'join', 'splitdrive', 'split', 'splite...
    altsep = None
    curdir = '.'
    defpath = ':/bin:/usr/bin'
    devnull = '/dev/null'
    extsep = '.'
    pardir = '..'
    pathsep = ':'
    sep = '/'
    supports_unicode_filenames = False


---

C++ 文件读写

fopen（）

fwrite（）

fread（）

[参考](http://blog.csdn.net/l740450789/article/details/49005325)

[ftell（）](http://zh.cppreference.com/w/c/io/ftell)

[fflush（）](http://c.biancheng.net/cpp/html/2506.html)

