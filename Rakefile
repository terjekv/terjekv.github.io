require "rubygems"
require "tmpdir"

require "bundler/setup"
require "jekyll"


# Change your GitHub reponame
GITHUB_REPONAME = "terjekv/terjekv.github.io"


desc "Generate blog files"
task :generate do
  Jekyll::Site.new(Jekyll.configuration({
    "source"      => ".",
    "destination" => "_site"
  })).process
end


desc "Generate and publish blog to master"
task :publish => [:generate] do
  system "cd _site"
  system "touch .nojekyll"
  system "git add . -A"
  message = "Site updated at #{Time.now.utc}"
  system "git commit -m #{message.inspect}"
  system "git push -u origin master"
end

