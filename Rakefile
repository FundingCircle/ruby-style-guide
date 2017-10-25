# frozen_string_literal: true

begin
  require 'rubygems'
  require 'bundler/setup'
rescue LoadError
  raise 'Bundler and Rubygems are required to run rake tasks for this project'
end

require 'rubocop/rake_task'
RuboCop::RakeTask.new

require 'rspec/core/rake_task'
RSpec::Core::RakeTask.new(:spec)

task default: %i[rubocop spec]
