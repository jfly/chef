#
# Author:: Adam Jacob (<adam@opscode.com>)
# Author:: Daniel DeLeo (<dan@opscode.com>)
# Copyright:: Copyright (c) 2008, 2010 Opscode, Inc.
# License:: Apache License, Version 2.0
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

VERSION = IO.read(File.expand_path("../VERSION", __FILE__)).strip

require 'rubygems'
require 'rubygems/package_task'
require 'rdoc/task'
require './tasks/rspec.rb'

GEM_NAME = "chef"

desc "build Gems of Chef's components"
task :package_components do
  Dir.chdir("chef-config") do
    sh "rake package"
  end
end

task :package => :package_components

desc "build and install chef's components"
task :install_components => :package_components do
  Dir.chdir("chef-config") do
    sh "rake install"
  end
end

task :install => :install_components

desc "clean up builds of Chef's components"
task :clobber_component_packages do
  Dir.chdir("chef-config") do
    sh "rake clobber_package"
  end
end

task :clobber_package => :clobber_component_packages

desc "Update the version number for Chef's components"
task :update_components_versions do
  Dir.chdir("chef-config") do
    sh "rake version"
  end
end

desc "Regenerate lib/chef/version.rb from VERSION file"
task :version => :update_components_versions do
  contents = <<-VERSION_RB
# Copyright:: Copyright (c) 2010-2015 Chef Software, Inc.
# License:: Apache License, Version 2.0
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

#!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
# NOTE: This file is generated by running `rake version` in the top level of
# this repo. Do not edit this manually. Edit the VERSION file and run the rake
# task instead.
#!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

class Chef
  CHEF_ROOT = File.dirname(File.expand_path(File.dirname(__FILE__)))
  VERSION = '#{VERSION}'
end

#
# NOTE: the Chef::Version class is defined in version_class.rb
#
# NOTE: DO NOT Use the Chef::Version class on Chef::VERSIONs.  The
#       Chef::Version class is for _cookbooks_ only, and cannot handle
#       pre-release chef-client versions like "10.14.0.rc.2".  Please
#       use Rubygem's Gem::Version class instead.
#
VERSION_RB
  version_rb_path = File.expand_path("../lib/chef/version.rb", __FILE__)
  IO.write(version_rb_path, contents)
end

Dir[File.expand_path("../*gemspec", __FILE__)].reverse.each do |gemspec_path|
  gemspec = eval(IO.read(gemspec_path))
  Gem::PackageTask.new(gemspec).define
end

desc "Build and install a chef gem"
task :install => [:package] do
  sh %{gem install pkg/#{GEM_NAME}-#{VERSION}.gem --no-rdoc --no-ri}
end

task :uninstall do
  sh %{gem uninstall #{GEM_NAME} -x -v #{VERSION} }
end

desc "Build it, tag it and ship it"
task :ship => [:clobber_package, :gem] do
  sh("git tag #{VERSION}")
  sh("git push opscode --tags")
  Dir[File.expand_path("../pkg/*.gem", __FILE__)].reverse.each do |built_gem|
    sh("gem push #{built_gem}")
  end
end

task :pedant do
  require File.expand_path('spec/support/pedant/run_pedant')
end

task :build_eventlog do
  Dir.chdir 'ext/win32-eventlog/' do
    system 'rake build'
  end
end

task :register_eventlog do
  Dir.chdir 'ext/win32-eventlog/' do
    system 'rake register'
  end
end

begin
  require 'yard'
  DOC_FILES = [ "README.rdoc", "LICENSE", "spec/tiny_server.rb", "lib/**/*.rb" ]
  namespace :yard do
    desc "Create YARD documentation"

    YARD::Rake::YardocTask.new(:html) do |t|
      t.files = DOC_FILES
      t.options = ['--format', 'html']
    end
  end

rescue LoadError
  puts "yard is not available. (sudo) gem install yard to generate yard documentation."
end

