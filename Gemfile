source "https://rubygems.org"
gem "minima", github: "jekyll/minima", ref: "38a84a9"
gem "github-pages", "~> 231", group: :jekyll_plugins

# Plugins
group :jekyll_plugins do
  gem "jekyll-feed"
end

gem "webrick"

# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", "~> 1.2"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]
