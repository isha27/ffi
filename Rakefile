require 'rubygems'
require 'rake/gempackagetask'
require 'rake/extensiontask'
require 'rubygems/specification'
require 'date'
require 'fileutils'
require 'rbconfig'

GEM = "ffi"
GEM_VERSION = "0.3.0"
AUTHOR = "Wayne Meissner"
EMAIL = "wmeissner@gmail.com"
HOMEPAGE = "http://kenai.com/projects/ruby-ffi"
SUMMARY = "A Ruby foreign function interface (compatible with Rubinius and JRuby FFI)"
LIBEXT = Config::CONFIG['host_os'].downcase =~ /darwin/ ? "dylib" : "so"
GMAKE = Config::CONFIG['host_os'].downcase =~ /bsd/ ? "gmake" : "make"
LIBTEST = "build/libtest.#{LIBEXT}"

spec = Gem::Specification.new do |s|
  s.name = GEM
  s.version = GEM_VERSION
  s.platform = Gem::Platform::RUBY
  s.has_rdoc = true
  s.extra_rdoc_files = ["README", "LICENSE"]
  s.summary = SUMMARY
  s.description = s.summary
  s.author = AUTHOR
  s.email = EMAIL
  s.homepage = HOMEPAGE
  s.rubyforge_project = 'ffi' 
  s.extensions = %w(ext/ffi_c/extconf.rb gen/Rakefile)
  
  s.require_path = 'lib'
  s.autorequire = GEM
  s.files = %w(LICENSE README Rakefile) + Dir.glob("{ext,lib,nbproject,samples,spec}/**/*")
end

Rake::ExtensionTask.new('ffi_c', spec) do |ext|
  ext.name = 'ffi_c'                # indicate the name of the extension.
  ext.ext_dir = 'ext'         # search for 'hello_world' inside it.
  ext.lib_dir = 'build'       # put binaries into this folder.
  ext.tmp_dir = 'build'         # temporary folder used during compilation.
  ext.cross_compile = true                # enable cross compilation (requires cross compile toolchain)
  ext.cross_platform = 'i386-mswin32'     # forces the Windows platform instead of the default one
end

TEST_DEPS = [ LIBTEST ]
if RUBY_PLATFORM == "java"
  desc "Run all specs"
  task :specs => TEST_DEPS do
    sh %{#{Gem.ruby} -S spec #{Dir["spec/ffi/*_spec.rb"].join(" ")} -fs --color}
  end
  desc "Run rubinius specs"
  task :rbxspecs => TEST_DEPS do
    sh %{#{Gem.ruby} -S spec #{Dir["spec/ffi/rbx/*_spec.rb"].join(" ")} -fs --color}
  end
else
  TEST_DEPS.unshift :compile
  desc "Run all specs"
  task :specs => TEST_DEPS do
    ENV["MRI_FFI"] = "1"
    sh %{#{Gem.ruby} -Ibuild -Ilib -S spec #{Dir["spec/ffi/*_spec.rb"].join(" ")} -fs --color}
  end
  desc "Run rubinius specs"
  task :rbxspecs => TEST_DEPS do
    ENV["MRI_FFI"] = "1"
    sh %{#{Gem.ruby} -Ibuild -Ilib -S spec #{Dir["spec/ffi/rbx/*_spec.rb"].join(" ")} -fs --color}
  end
end

Rake::GemPackageTask.new(spec) do |pkg|
  pkg.gem_spec = spec
end

desc "install the gem locally"
task :install => [:package] do
  sh %{sudo #{Gem.ruby} -S gem install pkg/#{GEM}-#{GEM_VERSION}}
end

desc "create a gemspec file"
task :make_spec do
  File.open("#{GEM}.gemspec", "w") do |file|
    file.puts spec.to_ruby
  end
end
task :clean do
  FileUtils.rm_rf("build")
  FileUtils.rm_rf("conftest.dSYM")
  FileUtils.rm_f(Dir["pkg/*.gem"])
end
task "build/libtest.#{LIBEXT}" do
  sh %{#{GMAKE} -f libtest/GNUmakefile}
end

desc "Test the extension"
task :test => [ :specs, :rbxspecs ] do

end
namespace :bench do
  ITER = ENV['ITER'] ? ENV['ITER'].to_i : 100000
  bench_libs = "-Ibuild -Ilib" unless RUBY_PLATFORM == "java"
  bench_files = Dir["bench/bench_*.rb"].reject { |f| f == "bench_helper.rb" }
  bench_files.each do |bench|
    task File.basename(bench, ".rb")[6..-1] => TEST_DEPS do
      sh %{#{Gem.ruby} #{bench_libs} #{bench} #{ITER}}
    end
  end
  task :all => TEST_DEPS do
    bench_files.each do |bench|
      sh %{#{Gem.ruby} #{bench_libs} #{bench}}
    end
  end
end
