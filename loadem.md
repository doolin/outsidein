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
gem 'jquery-rails'

group :assets do
  gem 'sass-rails',   '~> 3.2.3'
  gem 'coffee-rails', '~> 3.2.1'
  gem 'uglifier', '>= 1.0.3'
end

group :development do
  gem 'rdebug'
  gem 'pry-rails'
  gem 'pry-nav'
end
~~~~


`$ rake db:migrate`


# `application.rb` loads first

~~~~
@@@ ruby
module Loadem
  class Application < Rails::Application
    # Set Time.zone default to the specified zone and make Active
    # Run "rake -D time" for a list of tasks for finding time zone names. Default is UTC.
    config.time_zone = 'Central Time (US & Canada)'
    # Settings in config/environments/* take precedence over those specified here. 
  end
end
~~~~

# `config/environment.rb` is next loaded

Note we have to add the module explicitly:

~~~~
@@@ ruby
# Load the rails application
require File.expand_path('../application', __FILE__)

module Loadem
  class Application < Rails::Application
    config.time_zone = 'UTC'
  end
end
~~~~

\# Initialize the rails application
Loadem::Application.initialize!



# Specific environment is last loaded 

In `config/environments/development.rb` we have:

~~~~
@@@ ruby
Loadem::Application.configure do
  # ...
  config.time_zone = 'Pacific Time (US & Canada)' 
  # ...
end
~~~~




