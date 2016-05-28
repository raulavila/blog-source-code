GH_PAGES_DIR = "/Users/Raul/Work/Projects/GitHub/raulavila.github.io"
 
desc "Build Jekyll site and copy files"
task :build do
  system "jekyll build"
  system "rm -r #{GH_PAGES_DIR}/*"
  system "cp -r ./_site/* #{GH_PAGES_DIR}/"
end
