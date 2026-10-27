quillmarch_sources = [
  'main.c',
  'quillmarch-application.c',
  'quillmarch-window.c',
]

quillmarch_deps = [
  dependency('gtk4'),
  dependency('libadwaita-1', version: '>= 1.4'),
]

quillmarch_sources += gnome.compile_resources('quillmarch-resources',
  'quillmarch.gresource.xml',
  c_name: 'quillmarch'
)

executable('quillmarch', quillmarch_sources,
  dependencies: quillmarch_deps,
       install: true,
)
