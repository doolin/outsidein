
# Outside In with Cucumber & Friends

Cucumber is a great weapon in the arsenal of any web developer.  

Unfortunately, in mid-November 2010, most of the documentation for setting up Cucumber with Rails is for Rails 2 instead of Rails 3.  


This article helps fix a little bit of that...

# From scratch

This article and code is a from-scratch re-implementation of Sarah Mei's  <a href="http://www.sarahmei.com/blog/2010/05/29/outside-in-bdd/">Outside In BDD: How?</a> updated for Rails 3.  


But there are some differences.  Here, we start with a bare Rails application. The only generators we're going to use are for installing Cucumber.  Then, we'll drive the development one file and one method at a time. You will see lots of familiar error messages, along with exactly how those errors were fixed.


Here, we have a user adding a new book title to a list of book titles. That's all the information necessary to build out and test with Cucumber.


# Setting it up

First up, create your new Rails code:

~~~~
@@@ sh
$ rails new outsidein
$ cd outsidein
~~~~

Add <code>cucumber-rails</code> and <code>database_cleaner</code> 
to <code>:development</code> and <code>:test</code> groups in your Gemfile:

  
~~~~
@@@ ruby
source 'http://rubygems.org'

gem 'rails'
gem 'sqlite3'

gem 'sass-rails'
gem 'coffee-script'
gem 'uglifier'
gem 'jquery-rails'

group :test, :development do
  gem 'cucumber-rails'
  gem 'database_cleaner'
  gem 'rspec'
end
~~~~

# Bundle it

As usual, run bundler:

~~~~
@@@ sh
$ bundle install
  Fetching source index for http://rubygems.org/
  Using rake (0.9.2) 
  Using multi_json (1.0.3) 
  .
  .
  .
  Using turn (0.8.2) 
  Using uglifier (0.5.4) 
  Your bundle is complete! Use `bundle show [gemname]` to see where a bundled gem is installed.
$
~~~~


# Set up Cucumber
        
~~~~
@@@ sh
$ rails generate cucumber:install
   create  config/cucumber.yml
   create  script/cucumber
   chmod  script/cucumber
   create  features/step_definitions
   create  features/step_definitions/web_steps.rb
   create  features/support
   create  features/support/paths.rb
   create  features/support/selectors.rb
   create  features/support/env.rb
   exist  lib/tasks
   create  lib/tasks/cucumber.rake
   gsub  config/database.yml
   gsub  config/database.yml
   force  config/database.yml
$
~~~~

At this point, we're about ready to write our application.

# Step 1: Given I go to the new book page


Let's create our first feature, <code>features/book.feature</code>:

~~~~
@@@ gherkin
Feature: User manages books
  Scenario: User adds a new book
    Given I go to the new book page
    And I fill in "Name" with "War & Peace"
    When I press "Create" 
    Then I should be on the book list page
    And I should see "War & Peace"
~~~~


# We have no steps...


`$ cucumber`

We have no steps.
Solution: Add file <code>features/step_definitions/book_steps.rb</code>, and copy in the output:

~~~~
@@@ ruby
Given /^I go to the new book page$/ do
  pending # express the regexp above with the code you wish you had
end

Given /^I fill in "([^"]*)" with "([^"]*)"$/ do |arg1, arg2|
  pending # express the regexp above with the code you wish you had
end

When /^I press "([^"]*)"$/ do |arg1|
  pending # express the regexp above with the code you wish you had
end

Then /^I should be on the book list page$/ do
  pending # express the regexp above with the code you wish you had
end

Then /^I should see "([^"]*)"$/ do |arg1|
  pending # express the regexp above with the code you wish you had
end
~~~~


# Add action to node...

`$ cucumber`


~~~~
@@@ sh 
Using the default profile...
Feature: User manages books

  Scenario: User adds a new book            # features/book.feature:2
Deprecated: please use #source_tags instead.
    Given I go to the new book page         # features/step_definitions/book_steps.rb:1
      TODO (Cucumber::Pending)
      ./features/step_definitions/book_steps.rb:2:in `/^I go to the new book page$/'
      features/book.feature:3:in `Given I go to the new book page'
~~~~

Solution:

~~~~
@@@ ruby 
Given /^I go to the new book page$/ do
  visit new_book_path
end
~~~~


# Routing helps...

`$ cucumber`

We're failing at the first step of the scenario: <code>undefined local variable or method `new_book_path' for #<Cucumber::Rails::World:0x00000102ceac68> (NameError).</code>

Solution: Add <code>resources :books</code> to <code>config/routes.rb</code>.

While we're at it, go ahead and add a root path, this will be helpful 
later: `root :to => 'books#index'`.


# A route wants a controller...

`$ cucumber`


Failing again: <code>uninitialized constant BooksController (ActionController::RoutingError)</code> 


Solution: Add the controller file app/controllers/books_controller.rb

~~~~
@@@ ruby
class BooksController < ApplicationController
end
~~~~

# And controllers want actions

`$ cucumber`

Failing again: <code>The action 'new' could not be found for 
BooksController (AbstractController::ActionNotFound)</code>

Solution: Add the <code>new</code> method:

~~~~
@@@ ruby
class BooksController < ApplicationController
  def new
  end
end
~~~~


# Templates help too...


<dt>$ cucumber</dt>
<dd>Yikes! <code>Missing template books/new with {:handlers=>[:erb, :rjs, :builder, :rhtml, :rxml], :formats=>[:html], :locale=>[:en, :en]} in view paths "/Users/daviddoolin/src/bdd/app/views" (ActionView::MissingTemplate)</code>

~~~~
@@@ sh 
$ mkdir app/views/books
$ vi app/views/books/new.html.erb
~~~~

Just stick an <code>h2</code> in that file or something.

</dd>

<dt>$ cucumber</dt>
<dd>
Passed!  

One down, four to go.  
</dd>
</dl>


# Step 2: And I fill in "Name" with "War & Peace"


Cucumber now fails on the second step:


`$ cucumber`

~~~~
@@@ sh
And I fill in "Name" with "War & Peace" # features/step_definitions/book_steps.rb:5
  TODO (Cucumber::Pending)
  ./features/step_definitions/book_steps.rb:6:in `/^I fill in "([^"]*)" with "([^"]*)"$/'
  features/book.feature:4:in `And I fill in "Name" with "War & Peace"'
~~~~

Solution: add the <a href="https://github.com/jnicklas/capybara/blob/master/lib/capybara/node/actions.rb#L48">Capybara `fill_in` matcher</a>:

~~~~
@@@ ruby
Given /^I fill in "([^"]*)" with "([^"]*)"$/ do |arg1, arg2|
  fill_in(arg1, :with => arg2)
end
~~~~

</dd>


# Forms are very helpful

<dt>$ cucumber</dt> 
<dd>Fails: <code>cannot fill in, no text field, text area or password field with id, name, or label 'Name' found (Capybara::ElementNotFound)</code>

Solution: Add a form to the new book page:

~~~~
@@@ ruby
<%= form_for @book do |f| %>
  <%= f.label :name %>
  <%= f.text_field :name %>
  <%= f.submit 'Create' %>
<% end %>
~~~~
</dd>

# Rails is very unhappy

`$ cucumber`

Massive FAIL! <code>undefined method `model_name' for NilClass:Class (ActionView::Template::Error)</code>


Solution: Add instance variable to make Rails happy. <code>In app/controllers/books_controller.rb</code>, add <code>@book = Book.new</code>, like so:

~~~~
@@@ ruby
  def new
    @book = Book.new
  end
~~~~


# Instances prefer objects

<dt>$ cucumber</dt>
<dd>
Failing on <code>uninitialized constant BooksController::Book (NameError)</code>

Solution: Add model to make Rails happy: 

~~~~
@@@ sh
$ vi app/models/book.rb
~~~~

Make it look like this:

~~~~
@@@ ruby
class Book < ActiveRecord::Base
end
~~~~
</dd>

# Activate ActiveRecord

<dt>$ cucumber</dt>
<dd>
Failing Step 1 again... <code>Could not find table 'books' (ActiveRecord::StatementInvalid)</code>

Solution: First, create and edit a migration file:

~~~~
@@@ sh
$ mkdir db/migrate
$ vi db/migrate/20101120141414_create_books.rb
~~~~

Then create the migration:

~~~~
@@@ ruby
class CreateBooks < ActiveRecord::Migration
  def self.up
    create_table :books do |t|
      t.string :name

      t.timestamps
    end
  end
  def self.down
    drop_table :books
  end
end
~~~~

And run the migration:

~~~~
@@@ sh
 $ rake db:migrate
 $ rake db:test:prepare
~~~~


Running cucumber again, we pass.  Excellent.
</dd>

</dl>

# Step 3: When I press "Create"

<dl>
<dt>$ cucumber</dt>
<dd>
On to our next step:
<pre lang="sh">
When I press "Create"                   # features/step_definitions/book_steps.rb:9
   TODO (Cucumber::Pending)
   ./features/step_definitions/book_steps.rb:10:in `/^I press "([^"]*)"$/'
   features/book.feature:5:in `When I press "Create"'
</pre>

Solution: add the <a href="https://github.com/jnicklas/capybara/blob/master/lib/capybara/node/actions.rb#L34">Capybara `click_button` matcher</a>:

~~~~
@@@ ruby
When /^I press "([^"]*)"$/ do |arg1|
  click_button 'Create'
end
~~~~

</dd>

# Controllers love actions

`$ cucumber`

cucumber fails on action 'create': <code>The action 'create' could not be found for BooksController (AbstractController::ActionNotFound)</code>

Solution: Add the create method to the books controller:

~~~~
@@@ ruby 
def create
end
~~~~


# Another dang template

`$ cucumber`

Failing again on templates: <code>Missing template books/create with {:handlers=>[:erb</code>

Solution: We don't really want a "create" template, so 
let's go ahead and redirect this to the root_path for now:

~~~~
@@@ ruby
def create
  redirect_to books_path       
end
~~~~




`$ cucumber`

Failing and failing and failing: <code>The action 'index' could not be found for BooksController</code>.

Solution: Open <code>app/controllers/books_controller.rb</code>, add

~~~~
@@@ ruby
def index
end
~~~~


# An index action wants for an index template

`$ cucumber`

Bummer: <code>Missing template books/index with {:handlers=>[:erb</code>.

Solution: Add app/views/books/index.html.erb:

<pre>
&lt;h2>List books&lt;/h2>
</pre>

## Run it again to pass

`$ cucumber`


Step 3 now passes cucumber.  Onward, through the fog.


# Step 4: Then I should be on the book list page

<dl>

<dt>$ cucumber</dt>
<dd>
<pre lang="sh">
Then I should be on the book list page  # features/step_definitions/book_steps.rb:13
  TODO (Cucumber::Pending)
  ./features/step_definitions/book_steps.rb:14:in `/^I should be on the book list page$/'
  features/book.feature:6:in `Then I should be on the book list page'
</pre>
Time to fill in for the next step, this time with a matcher:
<pre lang="ruby">
Then /^I should be on the book list page$/ do
  page.should have_text('List books')
end
</pre>
</dd>

And that passes Step 4 (for now).

</dl>


# Step 5: And I should see "War & Peace"

<dl>


<dt>$ cucumber</dt>
<dd>
<pre lang="sh">
And I should see "War & Peace"       # features/step_definitions/book_steps.rb:17
  TODO (Cucumber::Pending)
  ./features/step_definitions/book_steps.rb:18:in `/^I should see "([^"]*)"$/'
  features/book.feature:7:in `And I should see "War & Peace"'
</pre>
Time to fill in for the next step, this time with a matcher:
<pre lang="ruby">
Then /^I should see "([^"]*)"$/ do |arg1|
  page.should have_text(arg1)
end
</pre>
</dd>

<dt>$ cucumber</dt>
<dd>
Not seeing books: <code>expected there to be text "War & Peace" in "List books" </code>.
Solution: Render the book list:
<pre>
$vi app/views/books/index.html.erb

&lt;h2>List books&lt;/h2>
  <%= render @books %>
</pre>
</dd>

<dt>$ cucumber</dt>
<dd>
Wooo... <code>undefined method `model_name' for NilClass:Class (ActionView::Template::Error)</code>.
Solution: Grab the list of books:
<pre>
def index
  @books = Book.all
end
</pre>
</dd>

# Rendering a partial requires... a partial

`$ cucumber`

Still failing... <code>Missing partial books/book with {:handlers=>[:erb,</code>.

Solution: Add the partial <code>app/views/books/_book.html.erb</code>

~~~~
@@@ ruby
< %= book.name %><br />
~~~~



# Time to actually create the book...

<dt>$ cucumber</dt>
<dd>
<code>expected #has_content?("War & Peace") to return true, got false </code>

Solution: We're almost done, add a little bit of code to the book controller's <code>create</code> method:

~~~~
@@@ ruby
  def create
    @book = Book.new(params[:book])
    if @book.save
      redirect_to root_path
    end
  end
~~~~

Here's what the entire controller class should look like now:

~~~~
@@@ ruby 
class BooksController < ApplicationController

  def index
    @books = Book.all
  end

  def new
    @book = Book.new
  end

  def create
    @book = Book.new(params[:book])
    if @book.save
      redirect_to root_path
    end
  end
end
~~~~

</dd>

</dl>



# And that's a wrap

Notes:
<ul>
<li>RSpec only for matchers. In the future (2013?), Capybara matchers may be sufficient.</li>
<li>All custom step definitions, no <code>web_steps.rb</code> matchers.
</li>
</ul>


# Conclusion

This isn't the only way to do this. Here are more references on the same topic:
<ul>
  <li><a href="http://rubylearning.com/blog/2010/10/05/outside-in-development/">Outside In Development</a> from Ruby Learning.</li>

  <li>Techiferous gives us <a href="http://techiferous.com/2010/04/using-capybara-in-rails-3/">Using Capybara in Rails 3</a>.</li>

  <li>John Wyles turns a pickle into a cucumber.</li>

  <li>Francis Fish handles <a href="http://www.francisfish.com/2010/03/05/debugging-cucumber-scripts-cucumber-and-devise-authentication">Devise with his cucumber</a>.</li>

</ul>

If you have an article you believe should be linked, let me know in the comments and I'll add it in.


Overall, this was a lot of work.  But there's more which could be done.  For example, deleting the <code>web_steps.rb</code> file, which is matching all the lines in the scenario, would force us to write our own matchers, and handle the testing code ourselves.

<strong>The entire project could be rewritten in RSpec alone, save the feature file.</strong>

<strong>What would you do? Did you give this 5-step procedure whirl?  Leave a note in the comments!</strong>



Enjoy!
<hr />