# Description: hello world example

licenses(["unencumbered"])  # public domain

exports_files(["LICENSE"])

load("@lisp__bazel//:bazel/rules.bzl", "lisp_binary")

lisp_binary(
    name = "hello",
    srcs = [
        "hello.lisp",
    ],
    main = "hello:main",
    visibility = ["//visibility:public"],
    deps = [
        "@lisp__alexandria//:alexandria",
    ],
)
