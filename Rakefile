require 'html-proofer'

task :default => :build
 
desc 'Build site with Jekyll.'
task :build  do
	print "Compiling website...\n"
  sh "bundle exec jekyll build"
  options = { :assume_extension => true, :external_only => true }
  HTMLProofer.check_directory("./_site", options).run
end
