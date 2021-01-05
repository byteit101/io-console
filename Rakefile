require "bundler/gem_tasks"
require "rake/testtask"

name = "io/console"

Rake::TestTask.new(:test) do |t|
  t.libs << "test" << "test/lib"
  t.libs << "lib"
  t.ruby_opts << "-rhelper"
  t.test_files = FileList["test/**/test_*.rb"]
end

require 'rake/extensiontask'
Rake::ExtensionTask.new(name)

task :default => [:compile, :test]

task "build" => "date_epoch"
task "date_epoch" do
  ENV["SOURCE_DATE_EPOCH"] = IO.popen(%W[git -C #{__dir__} log -1 --format=%ct], &:read).chomp
end

helper = Bundler::GemHelper.instance
def helper.update_gemspec
  path = "#{__dir__}/#{gemspec.name}.gemspec"
  File.open(path, "r+b") do |f|
    if (d = f.read).sub!(/^(_VERSION\s*=\s*)".*"/) {$1 + gemspec.version.to_s.dump}
      f.rewind
      f.truncate(0)
      f.print(d)
    end
  end
end

def helper.commit_bump
  sh(%W[git -C #{__dir__} commit -m bump\ up\ to\ #{gemspec.version}
        #{gemspec.name}.gemspec])
end

def helper.version=(v)
  gemspec.version = v
  update_gemspec
  commit_bump
  tag_version
end
major, minor, teeny = helper.gemspec.version.segments

task "bump:teeny" do
  helper.version = Gem::Version.new("#{major}.#{minor}.#{teeny+1}")
end

task "bump:minor" do
  helper.version = Gem::Version.new("#{major}.#{minor+1}.0")
end

task "bump:major" do
  helper.version = Gem::Version.new("#{major+1}.0.0")
end

task "bump" => "bump:teeny"
