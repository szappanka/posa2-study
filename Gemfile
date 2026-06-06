source "https://rubygems.org"

# Friss Jekyll 4.x — a github-pages gem régi Jekyll/Liquidjét szándékosan
# kerüljük, mert az új Rubyval összeférhetetlen (tainted? hiba).
gem "jekyll", "~> 4.3"

group :jekyll_plugins do
  # A .md fájlokra mutató linkeket (.md kiterjesztés nélkül) ez oldja fel.
  gem "jekyll-relative-links", "~> 0.7"
end

# Ruby 3.4+/4.0 alatt ezek kikerültek a stdlib alapcsomagból.
gem "webrick", "~> 1.8"
gem "csv"
gem "base64"
gem "bigdecimal"
