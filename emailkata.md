# An Email Code Kata for Rails

# 50,000 foot view

What we need:

* A `Mailer` (which is like a model)
* Email template (the view)
* An implementation, how the emails get sent.

# Some background

Sending emails from Rails isn't difficult, just takes a little practice to figure out and get working. 

Here is that practice, develped as a code kata.

* If possible, have a localhost email server running
* Consider using a temporary gemset
* Get a personal Sendgrid account (testing development)
* Set up Rails locally
* Create a heroku application, cedar stack.
* Add Sendgrid to the heroku application (testing production)
* Set up Rails email framework
* Configure SMTP for Sendgrid
* Drive out desired emails via test-first

# Development operations first


# Setting up

First, we need to set the stage:

~~~~
@@@ bash
$ rails new emailkata
$ cd emailkata
$ rm public/index.html
~~~~


# Build out the Gemfile

Now, add `haml` and `rspec` to the Gemfile:

~~~~
@@@ ruby
source 'https://rubygems.org'

gem 'rails', '3.2.1'
gem 'haml-rails'
gem 'jquery-rails'

group :assets do
  gem 'sass-rails',   '~> 3.2.3'
  gem 'coffee-rails', '~> 3.2.1'
  gem 'uglifier', '>= 1.0.3'
end

group :development, :test do
  gem 'sqlite3'
  gem 'rspec-rails'
  gem 'capybara'
  gem 'letter_opener'
  gem 'email_spec'
end
~~~~

We're moving the `sqlite3` gem into `:test` and `:development` groups to keep heroku cedar stack from sqawking at us.

## Don't forget to run the following:

~~~~
@@@ sh
$ bundle install
$ rails generate rspec:install
~~~~


# Do the git thing...

Let's go ahead and do the obvious next:

~~~~
@@@ sh
$ git init
$ git add .
$ git commit -m"first commit"
~~~~

We're going to need this next, when we fire up heroku.


# Heroku because we're going there anyway, probably

We want a new application on the `celadon cedar` stack. 

Do it like this:

`$ heroku apps:create --stack cedar`

New we can push to heroku: 

`$ git push heroku master`

# Building application email infrastructure


# What we're driving for

An application email infrastructure consists of the following:

* Web pages (views) which have links or forms for sending emails
* Web pages (views) which are redirect targets after email is sent
* Email templates (mailers), what the user gets sent.
* All the models, controllers, etc.

# Test driving the email setup


We really want to do this test-first, but `rails generate` saves a lot of time and hassle, so let's generate what we need to get started, then go from there.

~~~~
@@@ sh
$ rails generate mailer AlertMailer pop
      create  app/mailers/alert_mailer.rb
      invoke  haml
      create    app/views/alert_mailer
      create    app/views/alert_mailer/pop.text.haml
      invoke  rspec
      create    spec/mailers/alert_mailer_spec.rb
      create    spec/fixtures/alert_mailer/pop
~~~~

## RSpec it...

~~~~
@@@ sh
$ rspec spec/mailers
~~~~

### `git add .; git commit -m"generated mailer files"`


# What is an "ActionMailer", anyway?

From the [Rails ActionMailer API description](http://api.rubyonrails.org/classes/ActionMailer/Base.html),
we have this explanation (paraphrased):

> Emails are defined by creating methods within the model which are then used:
>
> 1. to set variables to be used in the mail template, 
> 2. to change options on the mail, or 
> 3. to add attachments.

Not too difficult, but as usual, there are a lot of moving parts.



# Starting into the email spec

Here's what we get for the `alert_mailer_spec.rb`:

~~~~
@@@ ruby 
require "spec_helper"

describe AlertMailer do
  describe "pop" do
    let(:mail) { AlertMailer.pop }

    it "renders the headers" do
      mail.subject.should eq("Pop")
      mail.to.should eq(["to@example.org"])
      mail.from.should eq(["from@example.com"])
    end

    it "renders the body" do
      mail.body.encoded.should match("Hi")
    end
  end
end
~~~~

Those parameters aren't useful. We want this emailer to actually work. I'm going
to use my real email address for testing it out. You should use your own.


# Controlling delivery

~~~~
@@@ sh
$ rails generate controller AlertPages sender thankyou
      create  app/controllers/alert_pages_controller.rb
       route  get "alert_pages/thankyou"
       route  get "alert_pages/sender"
      invoke  haml
      create    app/views/alert_pages
      create    app/views/alert_pages/sender.html.haml
      create    app/views/alert_pages/thankyou.html.haml
      invoke  rspec
      create    spec/controllers/alert_pages_controller_spec.rb
      create    spec/views/alert_pages
      create    spec/views/alert_pages/sender.html.haml_spec.rb
      create    spec/views/alert_pages/thankyou.html.haml_spec.rb
      invoke  helper
      create    app/helpers/alert_pages_helper.rb
      invoke    rspec
      create      spec/helpers/alert_pages_helper_spec.rb
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/alert_pages.js.coffee
      invoke    scss
      create      app/assets/stylesheets/alert_pages.css.scss
~~~~



# Bonus: why not `send` instead of `sender`?

Note we invoked generate with `sender` argument:

`$ rails generate controller AlertPages sender thankyou`

Why not `send` which is what we're intended to do?

# Run rspec

~~~~
@@@ ruby
$ rspec spec
..*..*

Pending:
  AlertPagesHelper add some examples to (or delete)
/private/tmp/emailkata/spec/helpers/alert_pages_helper_spec.rb
    # No reason given
    # ./spec/helpers/alert_pages_helper_spec.rb:14
  alert_pages/sender.html.haml add some examples to (or delete)
/private/tmp/emailkata/spec/views/alert_pages/sender.html.haml_spec.rb
    # No reason given
    # ./spec/views/alert_pages/sender.html.haml_spec.rb:4
  alert_pages/thankyou.html.haml add some examples to (or delete)
/private/tmp/emailkata/spec/views/alert_pages/thankyou.html.haml_spec.rb
    # No reason given
    # ./spec/views/alert_pages/thankyou.html.haml_spec.rb:4

Finished in 0.56446 seconds
~~~~


# Let's take a look at the routing first

And clean it up while we're at it.

This is what mine looks like now:

~~~~
@@@ ruby
Testemail::Application.routes.draw do
  get "alert_pages/sender"
  get "alert_pages/thankyou"
end
~~~~

Let's _not_ bother with setting up a `root` route. It's not necessary.

## And `rake routes` just for fun

~~~~
@@@ sh
alert_pages_sender GET /alert_pages/sender(.:format)   alert_pages#sender

alert_pages_thankyou GET /alert_pages/thankyou(.:format) alert_pages#thankyou
~~~~


We're going to use these later.



# Web pages supporting email

* Sender page
* Thank you page
* Helpers

# Sender page

This is the page where the user clicks to get an email sent. 

We're going to test for the link to send the email.

Add the following to `spec/views/alert_pages/sender.html.haml_spec.rb`:

~~~~
@@@ ruby

require 'spec_helper'
describe "alert_pages/sender" do
  it "has a send link on the page for sending email" do    
    render    
    rendered.should have_selector('a')
  end
end
~~~~

## Make it pass

~~~~
@@@ haml
%h1 AlertPages#sender
%p Find me in app/views/alert_pages/sender.html.haml
%p
  %a.alert-email{ :href => "#" } Send email
~~~~

# Thank you page

Add to `cat spec/views/alert_pages/thankyou.html.haml_spec.rb`:

~~~~
@@@ ruby 
require 'spec_helper'

describe "alert_pages/thankyou.html.haml" do
  it "thanks the user for sending him or herself email" do
    render
    rendered.should =~ /thank you/i
  end
end
~~~~

### `rspec spec` is red

## Go green

Add the following to `app/views/alert_pages/thankyou.html.haml`:

~~~~
@@@ haml
%h1 AlertPages#thankyou
%p Find me in app/views/alert_pages/thankyou.html.haml
%p Thank you for sending yourself email
%a { :href => alert_pages_sender_path }
~~~~


# Pages helper

~~~~
@@@ ruby
spec/helpers/alert_pages_helper_spec.rb 
require 'spec_helper'

describe AlertPagesHelper do
  it "provides a small footer element" do
    helper.footer_small.should =~ /small/
  end
end
~~~~

## And the implementation

~~~~
@@@ ruby
module AlertPagesHelper
  def footer_small
    'This is the small footer'
  end
end
~~~~

### Ok, that's kind of lame, I agree. But...

But but but...

The point is *testing first*.

It's a habit, and it's not an easy habit to create. 

If it were easy, everyone would do it.

# Do we even need a helper?

Maybe, maybe not.

Once you start thinking about split testing user behavior,
you need to think about how your code will support split testing.

At that point, helpers start to make a lot more sense.


# Controller specs

# We need to actually send emails...

This requires:

1. Invoking the `mail` method to acquire an email;
2. Delivering that email.

# Getting some help with specs

Using the handy [email_spec](https://github.com/bmabey/email-spec) gem,
put the following into `spec/spec_helper.rb`:

~~~~
@@@ ruby
require "email_spec"

RSpec.configure do |config|
  config.include(EmailSpec::Helpers)
  config.include(EmailSpec::Matchers)
end
~~~~

# Getting down and dirty with email specs...

Now we add in some of the handy spec with matchers from `email_spec` gem.

Add more to `spec/mailers/alert_mailer_spec.rb`:

~~~~
@@@ ruby
require "spec_helper"

describe AlertMailer do
  describe "pop" do
    # you already have this...
  end

  # from https://github.com/bmabey/email-spec
  before(:all) do
    @email = AlertMailer.pop
  end

  it "should be set to be delivered to the email passed in" do
    @email.should deliver_to("youremail@addresshere.com")
  end

  it "should contain the user's message in the mail body" do
    @email.should have_body_text(/Hi/)
  end

  it "should have the correct subject" do
    @email.should have_subject(/Pop/)
  end
end
~~~~

# Sending email

You have several options:

1. Run your own server
1. Leech from someone else's server
1. Take a chance with Google gmail smtp
1. Pay for smtp services such as Sendgrid

## Key point

''Ensure that your remote environment variables are set.''

This is a huge pain and will result in hours of entertainment
attempting to get all the pieces of the puzzle fit together properly.


# Email text view (with haml)


Check the email template, make sure it has something like this in
`app/views/alert_mailer/pop.text.haml`:

~~~~
@@@ haml
%p
  Your Email Alert

%p 
  #{@greeting}, find me in app/views/app/views/alert_mailer/pop.text.haml
~~~~



# Before going live with email...

Let's see what these emails look like in the browser first.

We'll use the handy `letter_opener` gem. Add the following to
`config/environments/development.rb`:

~~~~
@@@ ruby
Emailkata::Application.configure do
  # Settings here take precedence over config/application.rb
  ...
  config.action_mailer.delivery_method = :letter_opener
  #config.action_mailer.delivery_method = :smtp
  ...
end
~~~~

## Cool fact!

You don't have to be running Rails to use `letter_opener`.



# Going live with email

Up until now, we've been speccing out without having to actually run an email server. We've just had Rails doing it's Rails thing, and that's a good thing.

But now we make these emails actually get to where we want them.

1. localhost with postfix
2. personal Sendgrid account for development
3. Heroku managed Sendgrid account for production

Here's the command:

`heroku addons:add sendgrid:starter`

It should be possible to use the heroku managed Sendgrid 
locally on development. We'll check that out later using the 
heroku configuration command `heroku config`. Might even work for testing.


# Configure application email

Using the Sendgrid guidelines, emplace the following code into `config/environment.rb`:

~~~~
@@@ ruby
ActionMailer::Base.smtp_settings = {
  :user_name => ENV['SENDGRID_USERNAME'],
  :password => ENV['SENDGRID_PASSWORD'],
  :domain => "heroku.com",
  :address => "smtp.sendgrid.net",
  :port => 587,
  :authentication => :plain,
  :enable_starttls_auto => true
}
~~~~

If you use the environment variables `SENDGRID_USERNAME` 
and `SENDGRID_PASSWORD`, you will be able to leverage 
Heroku's automatically set environment variables. This is 
convenient for a number of reasons, mainly because it's convenient.



# Checklist

In approximate order of implementation:

1. App a spec for the mailer template file.
1. Add the email template to  `/app/mailers`.

For simple testing:

1. Add a view template spec checking for a link to an action which fires
   an email.
1. Add the view.
1. Add a controller spec on that action
1. Add the controller action.



# Troubleshooting

Setting up email has a number of moving parts. If you find yourself in a
bind, check the following:

* If you added Sendgrid via Heroku, ensure your environment variables are correct. 
*This is really important when testing Rails localhost and rake*. (See
note on rake below.)
* Precompiling assets may be necessary.
* `$ heroku logs` is your new best friend.

## When `rake` breaks...


It's really handy to run rake from the command line to test your emails,
like so:

~~~~
@@@ sh
$ rake dailynotice
~~~~

### IF...

1. you're using SendGrid, and,
2. you get a permission error...

...ensure you have your shell variables set in the shell session where
you are running rake:

~~~~
@@@ sh
export SENDGRID_USERNAME=myname
export SENDGRID_PASSWORD=mypass
~~~~



# Helpful links

* http://api.rubyonrails.org/classes/ActionMailer/Base.html
* https://github.com/bmabey/email-spec
* http://asciicasts.com/episodes/206-action-mailer-in-rails-3
* http://brianbruijn.com/?q=taxonomy/term/142
* https://github.com/sj26/mailcatcher
* https://github.com/ryanb/letter_opener



## Some good RSpec links

* http://highgroove.com/articles/2011/06/21/rspec-tip-let-and-nested-describe-context-blocks.html
* http://eggsonbread.com/2010/03/28/my-rspec-best-practices-and-tips/

- Action Mailer Screencast:
  http://rubyonrails.org/screencasts/rails3/bundler-action-mailer 
- Action Mailer Guide:
  http://guides.rubyonrails.org/action_mailer_basics.html
- Railscasts Screencast:
  http://railscasts.com/episodes/206-action-mailer-in-rails-3/
- Stanford Lectures:
  http://openclassroom.stanford.edu/MainFolder/HomePage.php
- Rails Apps Templates:
  https://github.com/RailsApps/rails3-bootstrap-devise-cancan#readme


# Postfix


~~~~
  618  postconf -n
  619  sudo launchctl load org.postfix.master.plist 
  620  sudo port install sendmail
  621  sudo port install mail
  622  sudo port install postfix
  623  sudo port uninstall postfix
  627  tail -f /var/log/mail.log
  628  date | mail -s test david.doolin@gmail.com
  629  tail -f /var/log/mail.log
  635  less /etc/postfix/main.cf
  636  sudo postfix stop
  637  sudo postfix start
  638  date | mail -s test david.doolin@gmail.com
  639  telnet localhost 25
  644  sudo less /System/Library/LaunchDaemons/org.postfix.master.plist
  645  sudo /bin/launchctl unload -w /System/Library/LaunchDaemons/org.postfix.master.plist
  646  tail -f /var/log/mail.log
  647  sudo port install postfix
  648  sudo port unload postfix
  649  tail -f /var/log/mail.log
  651  sudo /bin/launchctl load -w /System/Library/LaunchDaemons/org.postfix.master.plist
  652  tail -f /var/log/mail.log
  653  date | mail -s test david.doolin@gmail.com
~~~~
