# Zimtohrli https://github.com/google/zimtohrli
# Pinned to bca3c9cdaa505b2c1c2c4ef083c514db9f5f1d94, Apache-2.0 (LICENSE).
# Vendored verbatim: cpp/zimt/zimtohrli.h and cpp/zimt/zimtohrli.cc only.
# No FFT, audio I/O, tools, bindings, tests, or other upstream dependencies.
# Required dependency: Highway 1.4.0, gitlink
# 7fff3c667fb62475cdfb2b6c441452c8223c1a67, discovered as libhwy.pc.
# Standard math functions link libm where separate from the C++ runtime.
# Local build modifications only: this standalone Meson static-library build
# replaces upstream CMake; install zimt/zimtohrli.h and zimtohrli.pc.
# No fast-math: this test metric needs normal IEEE floating-point semantics.
# Intel builds use precise FP without SVML to avoid spurious FP traps.
# Portable C++17: no ISA flags, upstream Highway dispatch on supported targets;
# classic Intel and ARM32 are explicitly unsupported by this unmodified core:
# its float-to-double upper-half promotion cannot use Highway's scalar target,
# ARM32 NEON lacks doubles, and EMU128 is blocklisted by GCC < 16. MSVC-style
# compilers need _USE_MATH_DEFINES for upstream M_PI. Windows/macOS/mobile
# execution is left to platform CI; native GCC/Clang builds are validated.
project('zimtohrli', 'cpp', version: '0.0.0',
  default_options: ['cpp_std=c++17', 'b_staticpic=true'])

cpp = meson.get_compiler('cpp')
compile_args = []
if cpp.get_id() in ['intel', 'intel-cl'] or host_machine.cpu_family() == 'arm'
  error('Unmodified Zimtohrli core does not support classic Intel or ARM32; use MSVC, LLVM ICX, or a supported 64-bit target')
endif
if cpp.get_argument_syntax() == 'msvc'
  compile_args += ['-D_USE_MATH_DEFINES']
  compile_args += ['/fp:precise']
  if cpp.get_id().startswith('intel')
    compile_args += ['/Qimf-use-svml:false']
  endif
elif cpp.get_id() == 'intel'
  compile_args += ['-fp-model', 'precise', '-fimf-use-svml=false']
elif cpp.get_id().startswith('intel-llvm')
  compile_args += ['-fp-model=precise', '-fimf-use-svml=false', '-Wno-overriding-option']
else
  compile_args += cpp.get_supported_arguments('-fno-fast-math')
endif

hwy = dependency('libhwy', version: '>=1.4.0', static: true)
math_dep = cpp.find_library('m', required: false)
zimtohrli = static_library('zimtohrli',
  'cpp/zimt/zimtohrli.cc',
  include_directories: include_directories('cpp'),
  cpp_args: compile_args,
  dependencies: [hwy, math_dep],
  install: true)

install_headers('cpp/zimt/zimtohrli.h', subdir: 'zimt')
install_data('LICENSE', install_dir: get_option('datadir') / 'licenses/zimtohrli')

import('pkgconfig').generate(zimtohrli,
  filebase: 'zimtohrli',
  name: 'zimtohrli',
  description: 'Zimtohrli core perceptual audio metric')