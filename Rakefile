require 'rubygems'
require 'bundler/setup'

require 'rake/clean'

distro = nil
fpm_opts = ""

if File.exist?('/etc/system-release') && File.read('/etc/redhat-release') =~ /centos|redhat|fedora|amazon/i
  distro = 'rpm'
  fpm_opts << " --rpm-user root --rpm-group root "
elsif File.exist?('/etc/os-release') && File.read('/etc/os-release') =~ /ubuntu|debian/i
  distro = 'deb'
  fpm_opts << " --deb-user root --deb-group root "
end

unless distro
  $stderr.puts "Don't know what distro I'm running on -- not sure if I can build!"
end

version = '1.9.2'
release = Time.now.utc.strftime('%Y%m%d%H%M%S')
name = 'phantomjs'
description_string = %Q{PhantomJS is a headless WebKit scriptable with a JavaScript API. It has fast and native support for various web standards: DOM handling, CSS selector, JSON, Canvas, and SVG.}
jailed_root = File.expand_path('../jailed-root', __FILE__)
phantomjs_src_folder = File.expand_path('../phantomjs', __FILE__)

CLEAN.include jailed_root
CLEAN.include 'downloads'
CLEAN.include 'pkg'
CLEAN.include 'phantomjs'

task :init do
  mkdir_p File.join(jailed_root, '/opt/local/phantomjs')
  mkdir_p File.join(jailed_root, '/usr/bin')
  mkdir_p 'downloads'
  mkdir_p 'pkg'
end

task :download do
  cd 'downloads' do
    sh("curl --fail http://phantomjs.googlecode.com/files/phantomjs-#{version}-linux-x86_64.tar.bz2 > phantomjs-#{version}-linux-x86_64.tar.bz2 2>/dev/null")
    sh("echo 'c78c4037d98fa893e66fc516214499c58228d2f9  phantomjs-#{version}-linux-x86_64.tar.bz2'    > phantomjs-#{version}-linux-x86_64.tar.bz2.shasum 2>/dev/null")
    sh("sha1sum --status --check phantomjs-#{version}-linux-x86_64.tar.bz2.shasum")
  end
end

task :package do
  sh("tar jxf downloads/phantomjs-#{version}-linux-x86_64.tar.bz2 && mv phantomjs-#{version}-linux-x86_64/* #{jailed_root}/opt/local/phantomjs && rm -rf phantomjs-#{version}-linux-x86_64")
  ln_sf("../../opt/local/phantomjs/bin/phantomjs", "#{jailed_root}/usr/bin/phantomjs")
end

task :dist do
  cd "pkg" do
    sh(%Q{bundle exec fpm -s dir -t #{distro} --name #{name} -a x86_64 --version "#{version}" -C #{jailed_root} --verbose #{fpm_opts} --maintainer snap-ci@thoughtworks.com --vendor snap-ci@thoughtworks.com --url http://snap-ci.com --description "#{description_string}" --iteration #{release} --license 'https://github.com/ariya/phantomjs/blob/master/LICENSE.BSD' .})
  end
end

#build 2.0
#http://phantomjs.org/build.html
namespace :from_source do
  task :install_dependencies do
    sh %q{sudo yum -y install gcc gcc-c++ make flex bison gperf ruby openssl-devel freetype-devel fontconfig-devel libicu-devel sqlite-devel libpng-devel libjpeg-devel}
  end

  task :clone do
    sh %q{git clone --quiet git://github.com/ariya/phantomjs.git -b 2.0}
  end

  task :build do
    sh %q{cd phantomjs && (yes | ./build.sh)}
  end

  task :package do
    mkdir_p File.join(jailed_root, '/opt/local/phantomjs/bin')
    sh %Q{ mv #{phantomjs_src_folder}/bin/phantomjs #{File.join(jailed_root, '/opt/local/phantomjs/bin')}}
    ln_sf("../../opt/local/phantomjs/bin/phantomjs", "#{jailed_root}/usr/bin/phantomjs")
  end

  task :dist do
    cd "pkg" do
      sh(%Q{bundle exec fpm -s dir -t #{distro} --name phantomjs_2 -a x86_64 --version 2.0 -C #{jailed_root} --verbose #{fpm_opts} --maintainer snap-ci@thoughtworks.com --vendor snap-ci@thoughtworks.com --url http://snap-ci.com --description "#{description_string}" --iteration #{release} --license 'https://github.com/ariya/phantomjs/blob/master/LICENSE.BSD' .})
    end
  end

end

desc "build phantomjs rpm"
task default: [:clean, :init, :download, :package, :dist]

desc "build 2.0"
task build_20: [:clean, :init, 'from_source:install_dependencies', 'from_source:clone', 'from_source:build', 'from_source:package', 'from_source:dist']

desc 'build all'
task build_all: [:clean, :init, :download, :package, :dist, 'from_source:install_dependencies', 'from_source:clone', 'from_source:build', 'from_source:package', 'from_source:dist']