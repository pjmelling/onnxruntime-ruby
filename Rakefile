require "bundler/gem_tasks"
require "rake/testtask"

task default: :test
Rake::TestTask.new do |t|
  t.libs << "test"
  t.pattern = "test/**/*_test.rb"
end

shared_libraries = %w(libonnxruntime.so libonnxruntime.dylib onnxruntime.dll)

# ensure vendor files exist
task :ensure_vendor do
  shared_libraries.each do |file|
    raise "Missing file: #{file}" unless File.exist?("vendor/#{file}")
  end
end

Rake::Task["build"].enhance [:ensure_vendor]

def version
  "1.8.0"
end

def download_official(library, remote_lib, file, sha256)
  require "fileutils"
  require "open-uri"
  require "tmpdir"

  url = "https://github.com/microsoft/onnxruntime/releases/download/v#{version}/#{file}"
  puts "Downloading #{file}..."
  contents = URI.open(url).read

  computed_sha256 = Digest::SHA256.hexdigest(contents)
  raise "Bad hash: #{computed_sha256}" if computed_sha256 != sha256

  Dir.chdir(Dir.mktmpdir) do
    File.binwrite(file, contents)
    command = file.end_with?(".zip") ? "unzip -q" : "tar xf"
    system "#{command} #{file}"
    src = file[0..-5]
    dest = File.expand_path("vendor", __dir__)

    FileUtils.cp("#{src}/lib/#{remote_lib}", "#{dest}/#{library}")
    puts "Saved vendor/#{library}"

    if library.end_with?(".so")
      FileUtils.cp("#{src}/LICENSE", "#{dest}/LICENSE")
      puts "Saved vendor/LICENSE"
      FileUtils.cp("#{src}/ThirdPartyNotices.txt", "#{dest}/ThirdPartyNotices.txt")
      puts "Saved vendor/ThirdPartyNotices.txt"
    end
  end
end

# https://github.com/microsoft/onnxruntime/releases
namespace :vendor do
  task :linux do
    download_official("libonnxruntime.so", "libonnxruntime.so.#{version}", "onnxruntime-linux-x64-#{version}.tgz", "e299b5e8488ab345ad651cd010758f49fd31ad1fb8b2605f78ff199104029f17")
  end

  task :mac do
    download_official("libonnxruntime.dylib", "libonnxruntime.#{version}.dylib", "onnxruntime-osx-x64-#{version}.tgz", "5eacd50f98260f0664a9b9ca6ed091a25eb12646adfbfe29ccea8566345d58fe")
  end

  task :windows do
    download_official("onnxruntime.dll", "onnxruntime.dll", "onnxruntime-win-x64-#{version}.zip", "4860f27937911a14743df59620e019e508b4b595966ae6a3bc1ad2e3a72f7e6b")
  end

  task all: [:linux, :mac, :windows]

  task :platform do
    if Gem.win_platform?
      Rake::Task["vendor:windows"].invoke
    elsif RbConfig::CONFIG["host_os"] =~ /darwin/i
      Rake::Task["vendor:mac"].invoke
    else
      Rake::Task["vendor:linux"].invoke
    end
  end
end
