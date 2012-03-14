# Load 'em up

## A deeper look at the Rail boot sequence

# Rails new loadem

~~~~
$ rails new loadem

Add `rdebug` to the Gemfile.

~~~~
@@@ ruby
source 'https://rubygems.org'

gem 'rails', '3.2.1'

gem 'sqlite3'


group :assets do
  gem 'sass-rails',   '~> 3.2.3'
  gem 'coffee-rails', '~> 3.2.1'
  gem 'uglifier', '>= 1.0.3'
end

gem 'jquery-rails'

group :development do
  gem 'rdebug'
  gem 'pry-rails'
  gem 'pry-nav'
end
~~~~


`rake db:migrate`


