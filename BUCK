load('//:buckaroo_macros.bzl', 'buckaroo_deps')
load('//:subdir_glob.bzl', 'subdir_glob')

with_gflags = True

namespace = read_config('glog', 'namespace', 'google').strip()

genrule(
  name = 'config-h',
  out = 'config.h',
  srcs = [
    'src/config.h.cmake.in',
  ],
  cmd = 'awk \'{ gsub(/^#cmakedefine/, \"//cmakedefine\"); print; }\' < $SRCS > $OUT ',
)

genrule(
  name = 'gen-sh',
  out = 'gen.sh',
  cmd = r'''\
#!/bin/sh
cat > $OUT <<"EOF"
sed -e 's/@ac_cv_cxx_using_operator@/1/g' \
    -e 's/@ac_cv_have_unistd_h@/1/g' \
    -e 's/@ac_cv_have_stdint_h@/1/g' \
    -e 's/@ac_cv_have_systypes_h@/1/g' \
    -e 's/@ac_cv_have_libgflags@/{}/g' \
    -e 's/@ac_cv_have_uint16_t@/1/g' \
    -e 's/@ac_cv_have___builtin_expect@/1/g' \
    -e 's/@ac_cv_have_.*@/0/g' \
    -e 's/@ac_google_start_namespace@/namespace google {{/g' \
    -e 's/@ac_google_end_namespace@/}}/g' \
    -e 's/@ac_google_namespace@/google/g' \
    -e 's/@ac_cv___attribute___noinline@/__attribute__((noinline))/g' \
    -e 's/@ac_cv___attribute___noreturn@/__attribute__((noreturn))/g' \
    -e 's/@ac_cv___attribute___printf_4_5@/__attribute__((__format__ (__printf__, 4, 5)))/g'
EOF
'''.format(int(with_gflags)),
)

[
  genrule(
    name = f + '-h',
    srcs = [
      'src/glog/' + f + '.h.in',
    ],
    out = f + '.h',
    cmd = 'sh $(location :gen-sh) < $SRCS > $OUT',
  ) for f in [
    'vlog_is_on',
    'stl_logging',
    'raw_logging',
    'logging',
  ]
]

cxx_library(
  name = 'glog',
  header_namespace = '',
  exported_headers = {
    'glog/log_severity.h': 'src/glog/log_severity.h',
    'glog/vlog_is_on.h': ':vlog_is_on-h',
    'glog/stl_logging.h': ':stl_logging-h',
    'glog/raw_logging.h': ':raw_logging-h',
    'glog/logging.h': ':logging-h',
  },
  headers = dict(
    subdir_glob([
      ('src', '**/*.h'),
    ]).items() + [
      ('config.h', ':config-h'),
    ]
  ),
  srcs = glob([
    'src/demangle.cc',
    'src/logging.cc',
    'src/raw_logging.cc',
    'src/signalhandler.cc',
    'src/symbolize.cc',
    'src/utilities.cc',
    'src/vlog_is_on.cc',
  ]),
  platform_srcs = [
    ('windows.*', glob([ 'src/windows/**/*.cc' ])),
  ],
  deps = buckaroo_deps(),
  compiler_flags = [
    '-Wno-sign-compare',
    '-Wno-unused-function',
    '-Wno-unused-local-typedefs',
    '-Wno-unused-variable',
    '-DGLOG_BAZEL_BUILD',
    '-DGOOGLE_NAMESPACE=\'' + namespace + '\'',
  ],
  visibility = [
    'PUBLIC',
  ],
)
