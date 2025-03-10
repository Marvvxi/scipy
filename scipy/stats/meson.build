_stats_pxd = [
  fs.copyfile('__init__.py'),
  fs.copyfile('_stats.pxd'),
  fs.copyfile('_biasedurn.pxd'),
  fs.copyfile('_unuran/unuran.pxd'),
]

stats_special_cython_gen = generator(cython,
  arguments : cython_args,
  output : '@BASENAME@.c',
  depends : [
    _cython_tree,
    _ufuncs_pxi_pxd_sources,
    _stats_pxd,
    cython_special])

statlib_lib = static_library('statlib_lib',
  [
    'statlib/ansari.f',
    'statlib/spearman.f',
    'statlib/swilk.f'
  ],
  fortran_args: _fflag_Wno_unused_variable
)

statlib_module = custom_target('statlib_module',
  output: ['_statlib-f2pywrappers.f', '_statlibmodule.c'],
  input: 'statlib.pyf',
  command: [py3, generate_f2pymod, '@INPUT@', '-o', '@OUTDIR@']
)

statlib = py3.extension_module('_statlib',
  statlib_module,
  c_args: numpy_nodepr_api,
  fortran_args: fortran_ignore_warnings,
  link_with: statlib_lib,
  dependencies: [fortranobject_dep],
  include_directories: [inc_np, inc_f2py],
  link_args: version_link_args,
  install: true,
  link_language: 'fortran',
  subdir: 'scipy/stats'
)

_stats = py3.extension_module('_stats',
  stats_special_cython_gen.process('_stats.pyx'),
  c_args: cython_c_args,
  include_directories: [inc_np],
  link_args: version_link_args,
  install: true,
  subdir: 'scipy/stats'
)

mvn_module = custom_target('mvn_module',
  output: ['_mvn-f2pywrappers.f', '_mvnmodule.c'],
  input: 'mvn.pyf',
  command: [py3, generate_f2pymod, '@INPUT@', '-o', '@OUTDIR@']
)

mvn = py3.extension_module('_mvn',
  [mvn_module, 'mvndst.f'],
  c_args: numpy_nodepr_api,
  # Wno-surprising is to suppress a pointless warning with GCC 10-12
  # (see GCC bug 98411: https://gcc.gnu.org/bugzilla/show_bug.cgi?id=98411)
  fortran_args: [fortran_ignore_warnings, _fflag_Wno_surprising],
  dependencies: [fortranobject_dep],
  include_directories: [inc_np, inc_f2py],
  link_args: version_link_args,
  install: true,
  link_language: 'fortran',
  subdir: 'scipy/stats'
)

_sobol = py3.extension_module('_sobol',
  cython_gen.process('_sobol.pyx'),
  c_args: cython_c_args,
  include_directories: [inc_np],
  link_args: version_link_args,
  install: true,
  subdir: 'scipy/stats'
)

py3.install_sources([
    '_sobol_direction_numbers.npz'
  ],
  subdir: 'scipy/stats'
)

_qmc_cy = py3.extension_module('_qmc_cy',
  cython_gen_cpp.process('_qmc_cy.pyx'),
  cpp_args: cython_cpp_args,
  dependencies: thread_dep,
  include_directories: [inc_np],
  link_args: version_link_args,
  install: true,
  subdir: 'scipy/stats'
)

_stats_gen_pyx = custom_target('_stats_gen_pyx',
  output: [
    'beta_ufunc.pyx',  # 0 (used in stats/_boost - S_B)
    'binom_ufunc.pyx',  # 1 (S_B)
    'func_defs.pxd',  # 2 (S_B)
    'hypergeom_ufunc.pyx',  # 3 (S_B)
    'nbinom_ufunc.pyx',  # 4 (S_B)
    'templated_pyufunc.pxd',  # 5 (S_B)
    'ncf_ufunc.pyx',  # 6 (S_B)
    'ncx2_ufunc.pyx',  # 7 (S_B)
    'nct_ufunc.pyx',  # 8 (S_B)
    'skewnorm_ufunc.pyx',  # 9 (S_B)
    'invgauss_ufunc.pyx',  # 10 (S_B)
  ],
  input: '_generate_pyx.py',
  command: [py3, '@INPUT@', '-o', '@OUTDIR@'],
  depends: _stats_pxd
)

cython_stats_gen_cpp = generator(cython,
  arguments : cython_cplus_args,
  output : '@BASENAME@.cpp',
  depends : [_cython_tree, _stats_gen_pyx])

# Build recipes defined here to get correct output path when used from
# other subdirs.
beta_ufunc_pyx = cython_stats_gen_cpp.process(_stats_gen_pyx[0])
binom_ufunc_pyx = cython_stats_gen_cpp.process(_stats_gen_pyx[1])
hypergeom_ufunc_pyx = cython_stats_gen_cpp.process(_stats_gen_pyx[3])
nbinom_ufunc_pyx = cython_stats_gen_cpp.process(_stats_gen_pyx[4])
ncf_ufunc_pyx = cython_stats_gen_cpp.process(_stats_gen_pyx[6])
ncx2_ufunc_pyx = cython_stats_gen_cpp.process(_stats_gen_pyx[7])
nct_ufunc_pyx = cython_stats_gen_cpp.process(_stats_gen_pyx[8])
skewnorm_ufunc_pyx = cython_stats_gen_cpp.process(_stats_gen_pyx[9])
invgauss_ufunc_pyx = cython_stats_gen_cpp.process(_stats_gen_pyx[10])

# Extra dependency from _lib
unuran_wrap_pyx = lib_cython_gen.process('_unuran/unuran_wrapper.pyx')

biasedurn = py3.extension_module('_biasedurn',
  [
    cython_gen_cpp.process('_biasedurn.pyx'),
    'biasedurn/fnchyppr.cpp',
    'biasedurn/impls.cpp',
    'biasedurn/stoc1.cpp',
    'biasedurn/stoc3.cpp',
    'biasedurn/stocR.h',
    'biasedurn/wnchyppr.cpp',
  ],
  cpp_args: ['-DR_BUILD', cython_cpp_args],
  include_directories: [inc_np],
  link_args: version_link_args,
  # Note that we're not using npymath directly, it is needed to use npyrandom
  dependencies: [npyrandom_lib, npymath_lib],
  install: true,
  subdir: 'scipy/stats'
)

if use_pythran
  _stats_pythran_cpp = custom_target('_stats_pythran',
    output: ['_stats_pythran.cpp'],
    input: '_stats_pythran.py',
    command: [pythran, '-E', '@INPUT@', '-o', '@OUTDIR@/_stats_pythran.cpp']
  )

  _stats_pythran = py3.extension_module('_stats_pythran',
    _stats_pythran_cpp,
    cpp_args: cpp_args_pythran,
    include_directories: [incdir_pythran, incdir_numpy],
    link_args: version_link_args,
    install: true,
    subdir: 'scipy/stats'
  )
else
  py3.install_sources(
    ['_stats_pythran.py'],
    subdir: 'scipy/stats'
  )
endif


py3.install_sources([
    '__init__.py',
    '_axis_nan_policy.py',
    '_binned_statistic.py',
    '_binomtest.py',
    '_censored_data.py',
    '_common.py',
    '_constants.py',
    '_continuous_distns.py',
    '_covariance.py',
    '_crosstab.py',
    '_discrete_distns.py',
    '_distn_infrastructure.py',
    '_distr_params.py',
    '_entropy.py',
    '_fit.py',
    '_generate_pyx.py',
    '_hypotests.py',
    '_kde.py',
    '_ksstats.py',
    '_mannwhitneyu.py',
    '_morestats.py',
    '_mstats_basic.py',
    '_mstats_extras.py',
    '_multivariate.py',
    '_odds_ratio.py',
    '_page_trend_test.py',
    '_qmc.py',
    '_relative_risk.py',
    '_resampling.py',
    '_result_classes.py',
    '_rvs_sampling.py',
    '_sensitivity_analysis.py',
    '_stats_mstats_common.py',
    '_stats_py.py',
    '_tukeylambda_stats.py',
    '_variation.py',
    '_warnings_errors.py',
    'biasedurn.py',
    'contingency.py',
    'distributions.py',
    'kde.py',
    'morestats.py',
    'mstats.py',
    'mstats_basic.py',
    'mstats_extras.py',
    'mvn.py',
    'qmc.py',
    'sampling.py',
    'statlib.py',
    'stats.py'
  ],
  subdir: 'scipy/stats'
)

py3.install_sources([
    '_biasedurn.pxd',
    '_stats.pxd'
  ],
  subdir: 'scipy/stats'
)

py3.install_sources([
    '_qmc_cy.pyi',
    '_sobol.pyi'
  ],
  subdir: 'scipy/stats'
)

subdir('_boost')
subdir('_levy_stable')
subdir('_unuran')
subdir('_rcont')

subdir('tests')
