require 'rubygems'
require 'xcodebuild'
require 'cucumber/rake/task'

HERE = File.expand_path( '..',__FILE__ )
ENV['APP_BUNDLE_PATH'] = File.join( HERE, 'ci_artifacts/Frankified.app' )

namespace :xcode do
  XcodeBuild::Tasks::BuildTask.new :debug_simulator do |t|
    t.invoke_from_within = './app'
    t.configuration = "Debug"
    t.sdk = "iphonesimulator"
    t.formatter = XcodeBuild::Formatters::ProgressFormatter.new
  end
end

task :default => ["xcode:debug_simulator:cleanbuild"]

namespace :ci do
  def move_into_artifacts( src )
    FileUtils.mkdir_p( 'ci_artifacts' )
    FileUtils.mv( src, "ci_artifacts/" )
  end

  task :clear_artifacts do
    FileUtils.rm_rf( 'ci_artifacts' )
  end

  task :build => ["xcode:debug_simulator:cleanbuild"] do
    move_into_artifacts( Dir.glob("app/build/Debug-iphonesimulator/*.app") )
  end

  task :frank_build do
    sh '(cd app && frank build)'
    move_into_artifacts( "app/Frank/frankified_build/Frankified.app" )
  end
  
  # it's tricky to get the cropper tool to run properly from different working directories
  # so far, can only make it work if it's in same location as current working directory
  # when rake is running it's same directory as the file that this file exists in
  # when running Frank via cucumber, it's: app/Frank/
  # so, semi-brute force for now; try to extract it to both those locations
  # but also finding it tricky to use FileUtils to properly copy on top of existing dirs, and seems that .app sometimes behaves as a file, sometimes as a dir.
  # ... so, first removing, then exatrcing - every time, not pretty, but seems OK for now.
  task :extract_cropper do
    if (File.exists? "app/Frank/iOS-Simulator Cropper.app.zip")
      
      # remove existing unzipped versions. unzip again.
      if (File.directory? "iOS-Simulator Cropper.app") then
        FileUtils.remove_dir("iOS-Simulator Cropper.app")
      end
      sh 'unzip -o "app/Frank/iOS-Simulator Cropper.app.zip"'
      
      if (File.directory? "app/Frank/iOS-Simulator Cropper.app") then
        FileUtils.remove_dir("app/Frank/iOS-Simulator Cropper.app")
      end
      sh 'unzip -o "app/Frank/iOS-Simulator Cropper.app.zip" -d app/frank/'
    end
  end

  Cucumber::Rake::Task.new(:frank_test, 'Run Frank acceptance tests, generating HTML report as a CI artifact') do |t|
    t.cucumber_opts = "app/Frank/features --format pretty --format html --out ci_artifacts/frank_results.html"
  end
  
  task :move_screenshots do
    FileUtils.mv( Dir.glob('*.png'), "ci_artifacts/" )
  end

  def unzip(file, dest)
  unzip = Unzip.new
  unzip.destination = dest
  unzip.file = file
  unzip.force = true
  unzip.execute
  end
  
end

task :ci => ["ci:clear_artifacts","ci:build","ci:frank_build","ci:extract_cropper","ci:frank_test","ci:move_screenshots"]
