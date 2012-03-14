# An Email Code Kata for Rails



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


# Setting up

First, we need to set the stage:

~~~~
$ rails new emailkata
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

What we want a new application on the `celadon cedar` stack. 

Do it like this:

`$ heroku apps:create --stack cedar`

New we can push to heroku: 

`$ git push heroku master`



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
$ rails generate controller AlertEmailer sender thankyou
     create  app/controllers/alert_emailer_controller.rb
       route  get "alert_emailer/thankyou"
       route  get "alert_emailer/sender"
      invoke  haml
      create    app/views/alert_emailer
      create    app/views/alert_emailer/sender.html.haml
      create    app/views/alert_emailer/thankyou.html.haml
      invoke  rspec
      create    spec/controllers/alert_emailer_controller_spec.rb
      create    spec/views/alert_emailer
      create    spec/views/alert_emailer/sender.html.haml_spec.rb
      create    spec/views/alert_emailer/thankyou.html.haml_spec.rb
      invoke  helper
      create    app/helpers/alert_emailer_helper.rb
      invoke    rspec
      create      spec/helpers/alert_emailer_helper_spec.rb
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/alert_emailer.js.coffee
      invoke    scss
      create      app/assets/stylesheets/alert_emailer.css.scss
~~~~

## Bonus: why not `send` instead of `sender`?




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

~~~~
@@@ haml
%p
  Your StormSavvy Alert

%p 
  #{@greeting}, find me in app/views/app/views/alert_mailer/pop.text.haml
~~~~


# Going live with email


Up until now, we've been speccing out without having to actually run an email server. We've just had Rails doing it's Rails thing, and that's a good thing.

But now we make these emails actually get to where we want them.

1. localhost with postfix
2. personal Sendgrid account for development
3. Heroku managed Sendgrid account for production

Here's the command:

`heroku addons:add sendgrid:starter`

It should be possible to use the heroku managed Sendgrid locally on development. We'll check that out later using the heroku configuration command `heroku config`. Might even work for testing.

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

If you use the environment variables `SENDGRID_USERNAME` and `SENDGRID_PASSWORD`, you will be able to leverage Heroku's automatically set environment variables. This is convenient for a number of reasons, mainly because it's convenient.





# Troubleshooting

There are an countable infinity of ways to configure email systems. 


* If you added Sendgrid via Heroku, ensure your environment variables are correct.

* Precompiling assets may be necessary.

* `$ heroku logs` is your new best friend.



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


# Postfix


~~~~
  618  postconf -n
  619  sudo launchctl load org.postfix.master.plist 
  620  sudo port install sendmail
  621  sudo port install mail
  622  sudo port install postfix
  623  sudo port uninstall postfix
  624  sudo port uninstall postfix @2.8.5_0
  625  sudo port uninstall postfix @2.8.7_0
  626  sudo port uninstall postfix
  627  tail -f /var/log/mail.log
  628  date | mail -s test david.doolin@gmail.com
  629  tail -f /var/log/mail.log
  630* 
  631  tail -f /var/log/mail.log
  632  tail -f /var/log/mail.log
  633  tail -f /var/log/mail.log
  634  tail -f /var/log/mail.log
  635  less /etc/postfix/main.cf
  636  sudo postfix stop
  637  sudo postfix start
  638  date | mail -s test david.doolin@gmail.com
  639  telnet localhost 25
  640  telnet localhost 25
  641  telnet localhost 25
  642  less /System/Library/LaunchDaemons/org.postfix.master.plist 
  643  sudo /System/Library/LaunchDaemons/org.postfix.master.plist
  644  sudo less /System/Library/LaunchDaemons/org.postfix.master.plist
  645  sudo /bin/launchctl unload -w /System/Library/LaunchDaemons/org.postfix.master.plist
  646  tail -f /var/log/mail.log
  647  sudo port install postfix
  648  sudo port unload postfix
  649  tail -f /var/log/mail.log
  650  tail -f /var/log/mail.log
  651  sudo /bin/launchctl load -w /System/Library/LaunchDaemons/org.postfix.master.plist
  652  tail -f /var/log/mail.log
  653  date | mail -s test david.doolin@gmail.com
  654  history
~~~~