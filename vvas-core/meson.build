########################################################################
# Copyright (C) 2020-2022 Xilinx, Inc. 
# Copyright (C) 2022-2023 Advanced Micro Devices, Inc.
#########################################################################

project('vvas-libs-core', 'c', 'cpp',
  version : '1.0',
  meson_version : '>= 0.54.1',
  default_options : [ 'warning_level=1', 'cpp_std=c++14',
                      'buildtype=debugoptimized' ])

vvas_core_args = ['-DHAVE_CONFIG_H']

configinc = include_directories('.')
core_scaler_inc = include_directories('scaler')
core_overlay_inc = include_directories('overlay')
core_metaaffixer_inc = include_directories('metaaffixer')
core_decoder_inc = include_directories('decoder')
core_tracker_inc = include_directories('tracker')
core_parser_inc = include_directories('parser')
core_dpuinfer_inc = include_directories('dpuinfer')
core_metaconvert_inc = include_directories('metaconvert')

modules = []
project_version = meson.project_version()
version_arr = project_version.split('.')
core_version_major = version_arr[0].to_int()
core_version_minor = version_arr[1].to_int()

core_version = '@0@.@1@'.format(core_version_major, core_version_minor)
vvas_core_args = ['-DHAVE_CONFIG_H']

configinc = include_directories('.')
core_common_inc = include_directories('common')
core_utils_inc = [include_directories('utils')]

# Base Versioning Logic
soversion = 0
curversion = core_version_major * 100 + core_version_minor
libversion = '@0@.@1@.0'.format(soversion, curversion)

cdata = configuration_data()
cc = meson.get_compiler('cpp')

cdata.set_quoted('VVAS_CORE_VERSION', core_version)
cdata.set_quoted('VVAS_CORE_PACKAGE', 'vvas-lib-core')
cdata.set_quoted('VVAS_CORE_PACKAGE_VERSION', core_version)
cdata.set_quoted('VVAS_CORE_PACKAGE_NAME', 'VVAS Base libraries package')
cdata.set_quoted('VVAS_CORE_GETTEXT_PACKAGE', 'vvas-libs-core-' + core_version)

cdata.set_quoted('LIBDIR', join_paths(get_option('prefix'), get_option('libdir')))
cdata.set_quoted('LOCALEDIR', join_paths(get_option('prefix'), get_option('localedir')))

# Platform Checks
if host_machine.cpu_family() == 'x86_64'
  message ('compiling for PCIe platform')
  cdata.set('XLNX_PCIe_PLATFORM', 1)
elif host_machine.cpu_family() == 'aarch64'
  message ('compiling for Embedded platform')
  cdata.set('XLNX_EMBEDDED_PLATFORM ', 1)
else
  error('not supporting host machine families other than x86_64 & aarch64')
endif

# Logic for PPE and Utils
if get_option('enable_ppe') == 1
    cdata.set('ENABLE_PPE_SUPPORT ', 1)
endif

if get_option('vvas_core_utils') == 'GLIB'
  add_project_arguments('-DVVAS_GLIB_UTILS', language : ['c', 'cpp'])
endif

# Core Dependencies (Essential for xfilter/xmultisrc)
xrt_req = '>= 2.6'
xrt_dep = dependency('xrt', version : xrt_req, required : true)
uuid_dep = cc.find_library('uuid', required : true)
dl_dep = cc.find_library('dl', required : true)
pthread_dep = cc.find_library('pthread', required : true)

python3 = import('python').find_installation()
pkgconfig = import('pkgconfig')

# --- SUBDIRECTORY LOGIC (Gated by Options) ---

# Always needed for vvas_xfilter
subdir('utils')
subdir('common')

# Only build components if enabled
if get_option('dpuinfer').enabled()
    subdir('dpuinfer')
endif

if get_option('decoder').enabled()
    if get_option('pci_platform') == 'U30'
        cdata.set('XLNX_U30_PLATFORM ', 1)
        subdir('decoder_u30')
    else
        cdata.set('XLNX_V70_PLATFORM ', 1)
        subdir('decoder')
    endif
endif

if get_option('metaaffixer').enabled()
    subdir('metaaffixer')
endif

if get_option('overlay').enabled()
    subdir('overlay')
endif

if get_option('scaler').enabled()
    subdir('scaler')
endif

if get_option('tracker').enabled()
    subdir('tracker')
endif

if get_option('metaconvert').enabled()
    subdir('metaconvert')
endif

# Core infrastructure for plugins
subdir('parser')
subdir('pkgconfig')

# Final Config
vvas_config_h = configure_file(output : 'config.h', configuration : cdata)
install_headers(vvas_config_h, subdir : 'vvas_core/')