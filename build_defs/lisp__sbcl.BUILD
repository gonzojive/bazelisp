# Description: SBCL 1.x binary + source

VERSION="1.3.2" # Be sure to match your source code repository.
DEBUG_BUILD_SBCL=""

# Returns a list of sbcl contrib files, all under lib/sbcl/contrib.

CONTRIBS = [
    "sb-aclrepl",
    "sb-bsd-sockets",
    "sb-cltl2",
    "sb-concurrency",
    "sb-cover",
    "sb-executable",
    "sb-gmp",
    "sb-grovel",
    "sb-introspect",
    "sb-md5",
    "sb-posix",
    "sb-queue",
    "sb-rotate-byte",
    "sb-rt",
    "sb-simple-streams",
    "sb-sprof",
]

contribs_targets = \
    ([
         "lib/sbcl/contrib/asdf.fasl",
         "lib/sbcl/contrib/uiop.fasl",
     ] + ["lib/sbcl/contrib/%s.fasl" % x for x in CONTRIBS] +
     ["lib/sbcl/contrib/%s.asd" % x for x in CONTRIBS])

filegroup(
    name = "sbcl_contribs",
    srcs = contribs_targets,
)

# TODO: FIXME
# We need a copy of zlib named 'libz.a' to satisfy ld when SBCL's build
# tells it to look for -lz.
genrule(
    name = "libz",
    srcs = ["@c__zlib//:zlib"],
    outs = ["libz.a"],
    cmd = "a=($(locations @c__zlib//:zlib)) ; cp $${a[0]} $@",
    tools = ["@c__zlib//:zlib"],
)

genrule(
    name = "make-sbcl",
    srcs = [
        "@lisp__bazel//:make-sbcl-genrule",
        "libz.a",
        "@c__zlib//:headers",
        "@lisp__sbcl_binary_distribution//:sbcl",
        "@lisp__sbcl_binary_distribution//:sbcl-lib",
    ] + glob(["**"]),
    outs = contribs_targets + [
        "lib/libsbcl.a",
        "bin/sbcl",
        "lib/sbcl/sbcl.core",
    ],
    cmd = ("$(location @lisp__bazel//:make-sbcl-genrule) $(CC) \"$(CC_FLAGS)\"" +
           " $(location libz.a) '$(locations @c__zlib//:headers)'" +
           " $(location @lisp__sbcl_binary_distribution//:sbcl)" +
           " $(location :make.sh)" +
           " $(@D) " + VERSION + " " + DEBUG_BUILD_SBCL),
    visibility = ["//visibility:public"],
)

filegroup(
    name = "sbcl",
    srcs = [
        ":bin/sbcl",
    ],
    visibility = ["//visibility:public"],
)
filegroup(
    name = "sbcl-lib",
    srcs = [
        ":lib/sbcl/sbcl.core",
        ":sbcl_contribs",
    ],
    visibility = ["//visibility:public"],
)

# Make a file suitable for ld's --dynamic-list, listing all the
# symbols from sbcl. This allows only those symbols to be exported,
# not everything.
genrule(
    name = "libsbcl-exported-symbols",
    srcs = ["lib/libsbcl.a"],
    outs = ["libsbcl-exported-symbols.lds"],
    cmd = "nm -gp $< | " +
          "awk 'BEGIN {print \"{\"; } " +
          "/^[0-9a-fA-F].* . .*/ " +
          "{ gsub(\"^.* . \", \"\") ; printf \"  %s;\\n\", $$0 } " +
          "END { print \"};\" }' > $@",
    visibility = ["//visibility:public"],
)

cc_library(
    name = "libsbcl",
    srcs = ["lib/libsbcl.a"],
    linkopts = [
        # As it's difficult/impossible to tell blaze to include the
        # whole of a pre-built .a archive, I'll just mention some
        # symbols, as necessary, to ensure all necessary "unused" .o
        # files from the libsbcl.a archive actually do get included:
        "-Wl,--undefined=uid_username",  # wrap.o
        "-Wl,--undefined=lseek_largefile",  # largefile.o
        "-Wl,--undefined=get_timezone",  # time.o
        "-Wl,--undefined=spawn",  # run-program.o
        "-ldl",
        # TODO(jyknight): can this dependency be superseded by the
        # per-binary autogenerated "*.exportdynamic.lds" file now?
        # That list might be a superset of this, making this
        # redundant.
        "-Wl,--dynamic-list",
        "libsbcl-exported-symbols.lds",
    ],
    linkstatic = 1,
    visibility = ["//visibility:public"],
    deps = [
        "@c__zlib//:zlib",
        "libsbcl-exported-symbols.lds",
    ],
)

### Tests:

cc_binary(
    name = "smoketest-libsbcl-runtime",
    linkopts = ["-Wl,-no-pie"],
    deps = [
        "libsbcl",
        # TODO(bazel-team): this dependency should be transitive from the
        # libsbcl rule, but isn't. Without this extra dep, it cannot find the
        # .lds file
        "libsbcl-exported-symbols.lds",
    ],
)

genrule(
    name = "smoketest-libsbcl_rule",
    srcs = [
        ":smoketest-libsbcl-runtime",
        ":lib/sbcl/sbcl.core",
    ],
    outs = ["smoketest-libsbcl"],
    cmd = "$(location @lisp__bazel//:sbcl_support/combine-sbcl-image) $@"
    + " $(location :smoketest-libsbcl-runtime) $(location :lib/sbcl/sbcl.core)",
    executable = 1,
    tools = [
        "@lisp__bazel//:sbcl_support/combine-lisp-image",
    ],
)
