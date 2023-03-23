# SPDX-FileCopyrightText: 2023 Amyspark <amy@amyspark.me>
# SPDX-License-Identifier: BSD-3-Clause

project(
	'WavPack',
	'c', 'cpp',
	version: '5.6.2',
	license: 'BSD-3-Clause',
	default_options: [
		'c_std=c99',
		'warning_level=1',
	],
	meson_version: '>=0.60',
)

# keep in sync with configure.ac
LT_CURRENT = 3
LT_REVISION = 5
LT_AGE = 2

SOVERSION_MAJOR = LT_CURRENT - LT_AGE
SOVERSION_MINOR = LT_AGE
SOVERSION_MICRO = LT_REVISION

DYLIB_CURRENT_VERSION_MAJOR = LT_CURRENT + 1
DYLIB_CURRENT_VERSION_MINOR = LT_REVISION
DYLIB_CURRENT_VERSION_MICRO = 0
DYLIB_COMPATIBILITY_VERSION_MAJOR = LT_CURRENT + 1
DYLIB_COMPATIBILITY_VERSION_MINOR = 0
DYLIB_COMPATIBILITY_VERSION_MICRO = 0

DYLIB_CURRENT_VERSION = '@0@.@1@.@2@'.format(DYLIB_CURRENT_VERSION_MAJOR, DYLIB_CURRENT_VERSION_MINOR, DYLIB_CURRENT_VERSION_MICRO)
DYLIB_COMPATIBILITY_VERSION = '@0@.@1@.@2@'.format(DYLIB_COMPATIBILITY_VERSION_MAJOR, DYLIB_COMPATIBILITY_VERSION_MINOR, DYLIB_COMPATIBILITY_VERSION_MICRO)

compiler = meson.get_compiler('c')

# Languages

asm_enabled = compiler.get_argument_syntax() != 'msvc'
asm_att_enabled = asm_enabled
masm_enabled = add_languages('masm', required: compiler.get_id() == 'msvc')

# Options. See also dependent options below

enable_legacy = get_option('legacy')
enable_dsd = get_option('dsd')
install_docs = get_option('docs')

# Configuration tests

dependencies = []

m = compiler.find_library('m', required: false)

if m.found() and compiler.has_function('cos', dependencies: m)
	dependencies += m
endif

thread_dep = dependency('threads', required: false)
have_pthread_h = compiler.has_header('pthread.h', dependencies: thread_dep)
iconv_dep = dependency('iconv', required: false)
libcrypto_dep = dependency('libcrypto', required: false)

have_builtin_clz = compiler.links('''
	int main()
		{
			return __builtin_clz(1);
		}
	''',
	name: 'Supports __builtin_clz'
)

message('Checking large files support...')

HAVE_SYS_TYPES_H = compiler.has_header('sys/types.h')
HAVE_STDINT_H = compiler.has_header('stdint.h')
HAVE_STDDEF_H = compiler.has_header('stddef.h')
SIZEOF_OFF_T = compiler.sizeof('off_t')

large_files_supported = SIZEOF_OFF_T == 8
large_files_definitions = []

if not large_files_supported
	_large_files_definitions = '-D_FILE_OFFSET_BITS=64'
	message('Checking size of off_t with _FILE_OFFSET_BITS=64')
	SIZEOF_OFF_T = compiler.sizeof('off_t', args: _large_files_definitions)
	large_files_supported = SIZEOF_OFF_T == 8
	if large_files_supported
		_FILE_OFFSET_BITS = 64
		_FILE_OFFSET_BITS_CODE = '#define _FILE_OFFSET_BITS 64'
		large_files_definitions += '-D_FILE_OFFSET_BITS=64'
	endif
endif

if not large_files_supported
	_large_files_definitions += '-D_LARGEFILE_SOURCE'
	message('Checking size of off_t with _LARGEFILE_SOURCE')
	SIZEOF_OFF_T = compiler.sizeof('off_t', args: _large_files_definitions)
	large_files_supported = SIZEOF_OFF_T == 8
	if large_files_supported
		large_files_definitions += '-D_LARGEFILE_SOURCE'
	endif
endif

if not large_files_supported
	_large_files_definitions += '-D_LARGE_FILES'
	message('Checking size of off_t with _LARGE_FILES')
	SIZEOF_OFF_T = compiler.sizeof('off_t', args: _large_files_definitions)
	large_files_supported = SIZEOF_OFF_T == 8
	if large_files_supported
		large_files_definitions += '-D_LARGE_FILES'
	endif
endif

if large_files_supported
	if large_files_definitions.length() == 0
		message('Result of checking large files support: supported')
	else
		message('Result of checking large files support: supported with @0@'.format(
			' '.join(large_files_definitions)
			)
		)
		add_project_arguments(
			large_files_definitions,
			language: ['c', 'cpp'],
		)
	endif
else
	warning('Result of checking large files support: not supported')
endif

have_fseeko = compiler.has_function('fseeko')

have_masm = false
have_asm = false
cpu = target_machine.cpu_family()
is_windows = target_machine.system() == 'windows' or target_machine.system() == 'cygwin'

if is_windows and compiler.get_argument_syntax() == 'msvc'
	have_masm = masm_enabled and (cpu == 'x86_64' or cpu == 'i386')
else
	have_asm = asm_att_enabled and (cpu == 'x86_64' or cpu == 'i386' or cpu == 'arm')
endif

# Dependent options

enable_asm = get_option('asm').require(have_asm or have_masm).allowed()
enable_libcrypto = get_option('libcrypto').require(libcrypto_dep.found()).allowed()
build_programs = get_option('programs').require(is_windows or iconv_dep.found()).allowed()
build_cooledit_plugin = get_option('cooledit_plugin').require(is_windows).allowed()
build_winamp_plugin = get_option('winamp_plugin').require(is_windows).allowed()

# Targets

wavpack_srcs = files(
	'src/common_utils.c',
	'src/decorr_utils.c',
	'src/entropy_utils.c',
	'src/extra1.c',
	'src/extra2.c',
	'src/open_utils.c',
	'src/open_filename.c',
	'src/open_legacy.c',
	'src/open_raw.c',
	'src/pack.c',
	'src/pack_dns.c',
	'src/pack_floats.c',
	'src/pack_utils.c',
	'src/read_words.c',
	'src/tags.c',
	'src/tag_utils.c',
	'src/unpack.c',
	'src/unpack_floats.c',
	'src/unpack_seek.c',
	'src/unpack_utils.c',
	'src/write_words.c',
)

wavpack_headers = files(
	'include/wavpack.h',
)

if enable_legacy
	wavpack_srcs += files(
		'src/unpack3.c',
		'src/unpack3_open.c',
		'src/unpack3_seek.c',
	)
endif

if enable_dsd
	wavpack_srcs += files(
		'src/pack_dsd.c',
		'src/unpack_dsd.c',
	)
endif

if enable_asm
	if masm_enabled
		if cpu == 'i386'
			wavpack_srcs += [
				configure_file(
					input: 'src/pack_x86.asm',
					output: 'pack_x86.masm',
					copy: true,
				),
				configure_file(
					input: 'src/unpack_x86.asm',
					output: 'unpack_x86.masm',
					copy: true,
				),
				
			]
		elif cpu == 'x86_64'
			wavpack_srcs += [
				configure_file(
					input: 'src/pack_x64.asm',
					output: 'pack_x64.masm',
					copy: true,
				),
				configure_file(
					input: 'src/unpack_x64.asm',
					output: 'unpack_x64.masm',
					copy: true,
				),
				
			]
		endif
	elif asm_enabled
		if cpu == 'i386'
			wavpack_srcs += files(
				'src/pack_x86.S',
				'src/unpack_x86.S',
			)
		elif cpu == 'x86_64'
			wavpack_srcs += files(
				'src/pack_x64.S',
				'src/unpack_x64.S',
			)
		elif cpu == 'arm'
			wavpack_srcs += files(
				'src/unpack_armv7.S',
			)
		endif
	endif
endif

windows = import('windows')

winamp_rc = []

if target_machine.system() == 'windows'
	winamp_rc += windows.compile_resources(
		'winamp/wavpack.rc'
	)
endif

if target_machine.system() == 'windows' and get_option('default_library') != 'static'
	wavpack_srcs += windows.compile_resources(
		'wavpackdll/wavpackdll.rc'
	)
endif

wavpack_includes = [
	include_directories('include'),
	include_directories('src'),
]

wavpack_deps = [
	m,
	libcrypto_dep,
]

wavpack_defines = []

if enable_legacy
	wavpack_defines += '-DENABLE_LEGACY'
endif

if enable_dsd
	wavpack_defines += '-DENABLE_DSD'
endif

if compiler.get_argument_syntax() == 'msvc'
	wavpack_defines += '-D_CRT_SECURE_NO_WARNINGS'
endif

if have_builtin_clz
	wavpack_defines += '-DHAVE___BUILTIN_CLZ'
endif

if have_fseeko
	wavpack_defines += '-DHAVE_FSEEKO'
endif

if enable_libcrypto
	wavpack_defines += '-DHAVE_LIBCRYPTO'
endif

if enable_asm
	if cpu == 'i386'
		wavpack_defines += '-DOPT_ASM_X86'
	elif cpu == 'x86_64'
		wavpack_defines += '-DOPT_ASM_X64'
	elif cpu == 'arm'
		wavpack_defines += '-DOPT_ASM_ARM32'
	endif
endif

wavpack_masm_args = []

if enable_asm and have_masm and cpu == 'i386'
	wavpack_masm_args += '-safeseh'
endif

wavpack_link_args = []

if target_machine.system() == 'darwin'
	wavpack_link_args += '-Wl,-exported_symbols_list,@0@'.format(meson.current_source_dir() / 'libwavpack.sym')
	if get_option('default_library') != 'static'
		wavpack_link_args += compiler.get_supported_link_arguments(
			'-Wl,-compatibility_version,@0@'.format(DYLIB_COMPATIBILITY_VERSION),
			'-Wl,-current_version,@0@'.format(DYLIB_CURRENT_VERSION)
		)
	endif
elif target_machine.system() != 'windows' and target_machine.system() != 'cygwin'
	wavpack_link_args += '-Wl,--version-script=@0@'.format(meson.current_source_dir() / 'libwavpack.map')
	wavpack_link_args += compiler.get_supported_link_arguments(
		'-Wl,-no-undefined'
	)
endif

wavpack = library(
	'wavpack',
	wavpack_srcs,
	dependencies: wavpack_deps,
	include_directories: wavpack_includes,
	masm_args: wavpack_masm_args,
	c_args: wavpack_defines,
	link_args: wavpack_link_args,
	vs_module_defs: 'libwavpack.def',
	soversion: SOVERSION_MAJOR,
	version: '@0@.@1@.@2@'.format(SOVERSION_MAJOR, SOVERSION_MINOR, SOVERSION_MICRO),
	install: true,
)

install_headers(
	wavpack_headers,
	subdir: 'wavpack'
)

wavpack_dep = declare_dependency(
	link_with: wavpack,
	dependencies: wavpack_deps,
	include_directories: wavpack_includes,
)

if build_programs
	wavpack_programs_defines = [
		'-DPACKAGE_VERSION="@0@"'.format(meson.project_version()),
		'-DVERSION_OS="@0@"'.format(target_machine.system())
	]

	if compiler.get_argument_syntax() == 'msvc'
		wavpack_programs_defines += '-D_CRT_SECURE_NO_WARNINGS'
	endif

	wavpack_programs_includes = include_directories('cli')

	wavpackapp_srcs = files(
		'cli/wavpack.c',
		'cli/riff.c',
		'cli/wave64.c',
		'cli/aiff.c',
		'cli/caff.c',
		'cli/dsdiff.c',
		'cli/dsf.c',
		'cli/utils.c',
		'cli/md5.c',
		'cli/import_id3.c',
	)

	if is_windows
		wavpackapp_srcs += files(
			'cli/win32_unicode_support.c',
		)
	endif

	wavpackapp = executable(
		'wavpack',
		wavpackapp_srcs,
		c_args: wavpack_programs_defines,
		include_directories: wavpack_programs_includes,
		dependencies: [wavpack_dep, iconv_dep, m],
		install: true,
	)

	wvunpack_srcs = files(
		'cli/wvunpack.c',
		'cli/riff_write.c',
		'cli/wave64_write.c',
		'cli/aiff_write.c',
		'cli/caff_write.c',
		'cli/dsdiff_write.c',
		'cli/dsf_write.c',
		'cli/utils.c',
		'cli/md5.c',
	)

	if is_windows
		wvunpack_srcs += files(
			'cli/win32_unicode_support.c',
		)
	endif

	wvunpack = executable(
		'wvunpack',
		wvunpack_srcs,
		c_args: wavpack_programs_defines,
		include_directories: wavpack_programs_includes,
		dependencies: [wavpack_dep, iconv_dep, m],
		install: true,
	)

	wvgain_srcs = files(
		'cli/wvgain.c',
		'cli/utils.c',
	)

	if is_windows
		wvgain_srcs += files(
			'cli/win32_unicode_support.c',
		)
	endif

	wvgain = executable(
		'wvgain',
		wvgain_srcs,
		c_args: wavpack_programs_defines,
		include_directories: wavpack_programs_includes,
		dependencies: [wavpack_dep, m],
		install: true,
	)

	wvtag_srcs = files(
		'cli/wvtag.c',
		'cli/utils.c',
		'cli/import_id3.c',
	)

	if is_windows
		wvtag_srcs += files(
			'cli/win32_unicode_support.c',
		)
	endif

	wvtag = executable(
		'wvtag',
		wvtag_srcs,
		c_args: wavpack_programs_defines,
		include_directories: wavpack_programs_includes,
		dependencies: [wavpack_dep, iconv_dep, m,],
		install: true,
	)
endif

if build_cooledit_plugin
	cool_wv4_srcs = files(
		'audition/cool_wv4.c',
	)

	cool_wv4_srcs += winamp_rc

	cool_wv4_includes = [
		include_directories('audition'),
		include_directories('include'),
	]

	cool_wv4_defines = [
		'-DAUDITION_EXPORTS'
	]

	if compiler.get_argument_syntax() == 'msvc'
		cool_wv4_defines += '-D_CRT_SECURE_NO_WARNINGS'
	endif

	cool_wv4_link_args = []

	if compiler.get_id() == 'gcc' or compiler.get_id() == 'clang'
		cool_wv4_link_args += [
			'-mwindows',
			'-static-libgcc',
			'-Wl,--enable-stdcall-fixup',
		]
	endif

	cool_wv4 = shared_module(
		'cool_wv4',
		cool_wv4_srcs,
		c_args: cool_wv4_defines,
		link_args: cool_wv4_link_args,
		include_directories: cool_wv4_includes,
		name_suffix: 'flt',
		name_prefix: '',
		dependencies: wavpack_dep,
		vs_module_defs: 'audition/cool_wv4.def',
	)
endif

if build_winamp_plugin
	in_wv_srcs = files(
		'winamp/in_wv.c',
		'winamp/wasabi/wasabi.cpp',
	)

	in_wv_srcs += winamp_rc

	in_wv_includes = [
		include_directories('include'),
		include_directories('winamp'),
	]

	in_wv_defines = [
		'-DWINAMP_EXPORTS'
	]

	if compiler.get_argument_syntax() == 'msvc'
		in_wv_defines += '-D_CRT_SECURE_NO_DEPRECATE'
	endif

	in_wv_link_args = []

	if compiler.get_id() == 'gcc' or compiler.get_id() == 'clang'
		in_wv_link_args += [
			'-mwindows',
			'-static-libgcc',
			'-static-libstdc++',
		]
	endif

	in_wv = shared_module(
		'in_wv',
		in_wv_srcs,
		include_directories: in_wv_includes,
		c_args: in_wv_defines,
		cpp_args: in_wv_defines,
		link_args: in_wv_link_args,
		name_prefix: '',
		dependencies: wavpack_dep,
	)

	in_wv_lng_srcs = winamp_rc

	in_wv_lng_link_args = []

	if compiler.get_argument_syntax() == 'msvc'
		in_wv_lng_link_args += '-NOENTRY'
	endif

	if compiler.get_id() == 'gcc' or compiler.get_id() == 'clang'
		in_wv_lng_link_args += '-mwindows'
	endif

	in_wv_lng = shared_module(
		'in_wv_lng',
		in_wv_lng_srcs,
		link_args: in_wv_lng_link_args,
		name_suffix: 'lng',
		link_language: 'c',
	)
endif

# Installation

pkg = import('pkgconfig')

pkg.generate(
	wavpack,
	description: 'wavpack library',
)

if get_option('docs')
	wavpack_doc_names = files(
		'man/wavpack.1',
		'man/wvgain.1',
		'man/wvunpack.1',
		'man/wvtag.1',
	)

	install_man(
		wavpack_doc_names,
		install_dir: get_option('mandir') / 'man1'
	)
endif

# Tests

if get_option('tests') and have_pthread_h
	wvtest_srcs = files(
		'cli/wvtest.c',
		'cli/md5.c',
	)

	wvtest_includes = [
		include_directories('cli'),
	]

	if is_windows
		wvtest_srcs += files(
			'cli/win32_unicode_support.c',
		)

		wvtest_includes += include_directories('cli')
	endif

	wvtest = executable(
		'wvtest',
		wvtest_srcs,
		c_args: [
			'-DPACKAGE_VERSION="@0@"'.format(meson.project_version()),
			'-DVERSION_OS="@0@"'.format(target_machine.system())
		],
		dependencies: [wavpack_dep, thread_dep, m]
	)

	test(
		'wvtest',
		wvtest,
		args: ['--exhaustive', '--short', '--no-extras']
	)
endif
