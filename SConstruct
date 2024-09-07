#!/usr/bin/env python
import os
import shutil
# For reference:
# - CCFLAGS are compilation flags shared between C and C++
# - CFLAGS are for C-specific compilation flags
# - CXXFLAGS are for C++-specific compilation flags
# - CPPFLAGS are for pre-processor flags
# - CPPDEFINES are for pre-processor defines
# - LINKFLAGS are for linking flags

def normalize_path(val, env):
    return val if os.path.isabs(val) else os.path.join(env.Dir("#").abspath, val)


def validate_parent_dir(key, val, env):
    if not os.path.isdir(normalize_path(os.path.dirname(val), env)):
        raise UserError("'%s' is not a directory: %s" % (key, os.path.dirname(val)))


libname = "gdwiimote"
projectdir = "demo"

localEnv = Environment(tools=["default"], PLATFORM="")

customs = ["custom.py"]
customs = [os.path.abspath(path) for path in customs]

opts = Variables(customs, ARGUMENTS)
opts.Add(
    BoolVariable(
        key="compiledb",
        help="Generate compilation DB (`compile_commands.json`) for external tools",
        default=localEnv.get("compiledb", False),
    )
)
opts.Add(
    PathVariable(
        key="compiledb_file",
        help="Path to a custom `compile_commands.json` file",
        default=localEnv.get("compiledb_file", "compile_commands.json"),
        validator=validate_parent_dir,
    )
)
opts.Update(localEnv)

Help(opts.GenerateHelpText(localEnv))

env = localEnv.Clone()
env["compiledb"] = False

env.Tool("compilation_db")
compilation_db = env.CompilationDatabase(
    normalize_path(localEnv["compiledb_file"], localEnv)
)
env.Alias("compiledb", compilation_db)

env = SConscript("godot-cpp/SConstruct", {"env": env, "customs": customs})

# tweak this if you want to use different folders, or more folders, to store your source code in.
cpppath = ["src/"]
if env["platform"] == "windows":
    cpppath.append("C:\Program Files (x86)\WiiUse\include")
env.Append(CPPPATH=cpppath)

file = "{}{}{}".format(libname, env["suffix"], env["SHLIBSUFFIX"])

if env["platform"] == "macos":
    platlibname = "{}.{}.{}".format(libname, env["platform"], env["target"])
    file = platlibname

libraryfile = "bin/{}".format(file)
# env.Append(LIBS = ['wiiuse','bluetooth'])

wiiuselibname = "wiiuse"
if env["platform"] == "windows" and env["target"] == "template_debug":
    wiiuselibname = "wiiuse_debug"

libs = [wiiuselibname]
libpath = ["bin/"]

if env["platform"] == "linux":
    libs.append("bluetooth")

if env["platform"] == "windows":
    libpath.append("C:\Program Files (x86)\WiiUse\lib")

env.Append(LIBPATH=libpath)
env.Append(LIBS=libs)

sources = Glob("src/*.cpp")

library = env.SharedLibrary(
    libraryfile,
    source=sources,
)

prefix=""
if env["platform"] == "linux":
    prefix = "lib"


copy_to_project = env.Command(f"{projectdir}/bin/{prefix}{file}", f"bin/{prefix}{file}", Copy("$TARGET", "$SOURCE"))

env.Depends(copy_to_project, library)

default_args = [library, copy_to_project]
if localEnv.get("compiledb", False):
    default_args += [compilation_db]
Default(*default_args)
