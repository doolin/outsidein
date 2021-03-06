
# Outside In with Cucumber & Friends

Cucumber is a great weapon in the arsenal of any web developer.  

Unfortunately, in mid-November 2010 (when this material was first written),
most of the documentation for setting up Cucumber with Rails is for Rails 2 instead of Rails 3.  

This article helps fix a little bit of that.


#### Disclaimer

Best Practice in Cucumber has evolved considerably since
this article was first written. 

Just to be clear, the purpose of the exercise here is:

1. Demonstrate what the Outside-In cycle looks like
1. Demonstrate approximately how test-first is implemented

Again: nothing in this presentation should be construed
as a BDD Best Practice.


# 1. From scratch

This article and code is a from-scratch re-implementation of Sarah Mei's  <a href="http://www.sarahmei.com/blog/2010/05/29/outside-in-bdd/">Outside In BDD: How?</a> updated for Rails 3.  


But there are some differences.  Here, we start with a bare Rails application. The only generators we're going to use are for installing Cucumber.  Then, we'll drive the development one file and one method at a time. You will see lots of familiar error messages, along with exactly how those errors were fixed.

Here, we have a user adding a new book title to a list of book titles. That's all the information necessary to build out and test with Cucumber.

### With a little help from your friends...

Try this round robin-robin style with one or more friends. One person
starts with the setup, pushes to github. Everyone else pulls to
update, then leadeship passes to the next person, who implements
the next step and pushes. Repeat until done.

## Assumptions

I'm using the following setup:

 * ruby 1.9.3-p0
  * `rvm use 1.9.3@outsidein`
 * rails 3.2.1
 * Gemfile to follow...
 
# 2. Setting it up

First up, create your new Rails code:

~~~~
@@@ sh
$ rails new outsidein
$ cd outsidein
$ rm public/index.html
~~~~

## OpenSSL error

If bundler segfaults, this is most likely a
problem with the `openssl` library which it 
was compiled against. 

For now, change the source argument from `https` to
`http` in the Gemfile


# 3. Building the Gemfile

Add <code>cucumber-rails</code>, `rspec` and <code>database_cleaner</code> 
to <code>:development</code> and <code>:test</code> groups in your Gemfile:

  
~~~~
@@@ ruby
source 'http://rubygems.org'

gem 'rails', '3.2.1'
gem 'sqlite3'

group :assets do
  gem 'sass-rails'
  gem 'coffee-script' 
  gem 'uglifier'
end

gem 'jquery-rails'

group :test, :development do
  gem 'cucumber-rails'
  gem 'database_cleaner'
  gem 'rspec'
  gem 'spork' #optional
end
~~~~

# 4. Bundle it

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


# 5. Set up Cucumber
        
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

# 6. Step 1: Given I go to the new book page


Let's create our first feature. In <code>features/</code>, create a file called <code>book.feature</code>:

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


### Caveat: This is feature is somewhat brittle!

Current best practice deprecates features which 
specify form filling.

# 7. We have no steps...


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


# 8. Add action to node...

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


# 9. Routing helps...

`$ cucumber`

We're failing at the first step of the scenario: <code>undefined local variable or method `new_book_path' for #<Cucumber::Rails::World:0x00000102ceac68> (NameError).</code>

Solution: Add <code>resources :books</code> to <code>config/routes.rb</code>.

While we're at it, go ahead and add a root path, this will be helpful 
later: `root :to => 'books#index'`.

## Important: If you're running Spork...

If you're running Spork, you will need to restart rails to 
acquire the reconfigured routes.

# 10. A route wants a controller...

`$ cucumber`


Failing again: <code>uninitialized constant BooksController (ActionController::RoutingError)</code> 


Solution: Add the controller file app/controllers/books_controller.rb

~~~~
@@@ ruby
class BooksController < ApplicationController
end
~~~~

# 11. And controllers want actions

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


# 13. Templates help too...


`$ cucumber`

<code>Missing template books/new with {:handlers=>[:erb, :rjs, :builder, :rhtml, :rxml], :formats=>[:html], :locale=>[:en, :en]} in view paths "/Users/daviddoolin/src/bdd/app/views" (ActionView::MissingTemplate)</code>

~~~~
@@@ sh 
$ mkdir app/views/books
$ vi app/views/books/new.html.erb
~~~~

Just stick an `h1` in that file or something:

~~~~
@@@ html
<h1>New book page<h1>
~~~~


## Run it again to pass

`$ cucumber`

Passed!  

One down, four to go.  


# 14. Step 2: And I fill in "Name" with "War & Peace"


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



# 15. Forms are very helpful

`$ cucumber`

Fails: <code>cannot fill in, no text field, text area or password field with id, name, or label 'Name' found (Capybara::ElementNotFound)</code>

Solution: Add a form to the new book page:

~~~~
@@@ ruby
<%= form_for @book do |f| %>
  <%= f.label :name %>
  <%= f.text_field :name %>
  <%= f.submit 'Create' %>
<% end %>
~~~~


# 16. Rails is very unhappy

`$ cucumber`

Massive FAIL! <code>undefined method `model_name' for NilClass:Class (ActionView::Template::Error)</code>


Solution: Add instance variable to make Rails happy. <code>In app/controllers/books_controller.rb</code>, add <code>@book = Book.new</code>, like so:

~~~~
@@@ ruby
  def new
    @book = Book.new
  end
~~~~


# 17. Instances prefer objects

`$ cucumber`

Failing on <code>uninitialized constant BooksController::Book (NameError)</code>.

Solution: This is a somewhat confusing error messge, we need a model to make Rails happy:

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


# 18. Activate ActiveRecord

`$ cucumber`

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


# 19. Step 3: When I press "Create"


`$ cucumber`

On to our next step:

~~~~
@@@ sh 
When I press "Create"                   # features/step_definitions/book_steps.rb:9
   TODO (Cucumber::Pending)
   ./features/step_definitions/book_steps.rb:10:in `/^I press "([^"]*)"$/'
   features/book.feature:5:in `When I press "Create"'
~~~~

Solution: add the <a href="https://github.com/jnicklas/capybara/blob/master/lib/capybara/node/actions.rb#L34">Capybara `click_button` matcher</a>:

~~~~
@@@ ruby
When /^I press "([^"]*)"$/ do |arg1|
  click_button 'Create'
end
~~~~



# 20. Controllers love actions

`$ cucumber`

cucumber fails on action 'create': <code>The action 'create' could not be found for BooksController (AbstractController::ActionNotFound)</code>

Solution: Add the create method to the books controller:

~~~~
@@@ ruby 
def create
end
~~~~


# 21. Another dang template

`$ cucumber`

Failing again on templates: <code>Missing template books/create with {:handlers=>[:erb</code>

Solution: We don't really want a "create" template, so 
let's go ahead and redirect this to the root_path for now:

~~~~
@@@ ruby
def create
  redirect_to root_path
end
~~~~


# 22. Handle the index action...

`$ cucumber`

Failing and failing and failing: <code>The action 'index' could not be found for BooksController</code>.

Solution: Open <code>app/controllers/books_controller.rb</code>, add

~~~~
@@@ ruby
def index
end
~~~~


# 23. An index action wants for an index template

`$ cucumber`

Bummer: <code>Missing template books/index with {:handlers=>[:erb</code>.

Solution: Add app/views/books/index.html.erb:

~~~~
@@@ html
<h2>List books</h2>
~~~~

## Run it again to pass

`$ cucumber`

Step 3 now passes cucumber.  Onward, through the fog.


# 24. Step 4: Then I should be on the book list page

`$ cucumber`

~~~~
@@@ sh
Then I should be on the book list page  # features/step_definitions/book_steps.rb:13
  TODO (Cucumber::Pending)
  ./features/step_definitions/book_steps.rb:14:in `/^I should be on the book list page$/'
  features/book.feature:6:in `Then I should be on the book list page'
~~~~

Time to fill in for the next step, this time with a matcher:

~~~~
@@@ ruby
Then /^I should be on the book list page$/ do
  page.should have_content('List books')
end
~~~~

## Run it again to pass

`$ cucumber`

And that passes Step 4.

</dl>


# 25. Step 5: And I should see "War & Peace"



`$ cucumber`


~~~~
@@@ sh
And I should see "War & Peace"       # features/step_definitions/book_steps.rb:17
  TODO (Cucumber::Pending)
  ./features/step_definitions/book_steps.rb:18:in `/^I should see "([^"]*)"$/'
  features/book.feature:7:in `And I should see "War & Peace"'
~~~~

Time to fill in for the next step, this time with a matcher:

~~~~
@@@ ruby
Then /^I should see "([^"]*)"$/ do |arg1|
  page.should have_content(arg1)
end
~~~~


# 26. Still not seeing any books

`$ cucumber`

Not seeing books: <code>expected there to be text "War & Peace" in "List books" </code>.

Solution: Render the book list. First, open the template file:

~~~~
@@@ sh
$vi app/views/books/index.html.erb
~~~~

Now render the books:

~~~~
@@@ ruby
<h2>List books</h2>
  <%= render @books %>
~~~~



# 27. Need an instance array of books

`$ cucumber`

<code>undefined method `model_name' for NilClass:Class (ActionView::Template::Error)</code>.

Solution: Grab the list of books:

~~~~
@@@ ruby
def index
  @books = Book.all
end
~~~~


# 28. Rendering a partial requires... a partial

`$ cucumber`

Still failing... <code>Missing partial books/book with {:handlers=>[:erb,</code>.

Solution: Add the partial <code>app/views/books/_book.html.erb</code>

~~~~
@@@ ruby
<%= book.name %>
~~~~



# 29. Time to actually create the book...

`$ cucumber`

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


# 30. Run it again to pass

`$ cucumber`

We're done.

## And that's a wrap

Notes:

* RSpec only for matchers. In the future (2013?), Capybara matchers may be sufficient.
* All custom step definitions, no <code>web_steps.rb</code> matchers.



# Conclusion

This isn't the only way to do this. Here are more references on the same topic:
<ul>
  <li><a href="http://rubylearning.com/blog/2010/10/05/outside-in-development/">Outside In Development</a> from Ruby Learning.</li>

  <li>Techiferous gives us <a href="http://techiferous.com/2010/04/using-capybara-in-rails-3/">Using Capybara in Rails 3</a>.</li>

  <li>John Wyles turns a pickle into a cucumber.</li>

  <li>Francis Fish handles <a href="http://www.francisfish.com/2010/03/05/debugging-cucumber-scripts-cucumber-and-devise-authentication">Devise with his cucumber</a>.</li>

</ul>

If you have an article you believe should be linked, let me know in the comments and I'll add it in.


Overall, this was a lot of work.  But there's more which could be done.  For example:

<strong>The entire project could be rewritten in RSpec alone, save the feature file.</strong>

<strong>What would you do? Did you give this 5-step procedure whirl?  Leave a note in the comments!</strong>



Enjoy!
<hr />
