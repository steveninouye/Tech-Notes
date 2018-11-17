# Tips and Tricks for Assessment A04

I really want all of us to graduate together so here are a few tips and tricks to get faster in creating a Rails project. This will start off slow but each will build up on each other. Edits will be made as more is found.

If you see something in all capital letters, replace it to corresponding  word/classname/command

## Connect Your Models & Migrations In 1 Command

### Level 1

You already know that if you're going to make a migration, you are going to make a model. You can create a model and migration in 1 command in your terminal.

```bash
# remember that models are singular

bundle exec rails g model MODELNAME

## bundle exec rails g model User
```
After running this command will make your model file and migration file along with a nice skeleton.

### Level 2
You are able to have rails  add your column names into your migration files for you with the same command. Do this by passing the command arguments. Specify the data type for the column by adding a colon followed by the data type.

```bash
bundle exec rails g model MODEL NAME COLUMNNAME1:DATA-TYPE COLUMNNAME2:DATA-TYPE COLUMNNAME3:DATA-TYPE

## bundle exec rails g model User username:string age:integer password_digest:string session_token:string
```
### Level 3

The data type defaults to string so you could simplify the above example by
```bash
bundle exec rails g model User username age:integer password_digest session_token
```
*You must add your own index and constraints to any columns you need them on*

## Create Your Controller and Views in One Command
### Level 1
When you create a controller you know you are going to create views for it. You are able to set up a skeleton controller file and the directory corresponding to that controller in a single command.
```bash
# do not add "Controller" to the controller name
#remember that controllers are plural

bundle exec rails g controller CONTROLLERNAME

## bundle exec rails g controller Users
```

## Level 2
You can add your methods in your controller with the same command. All you need to do is pass it corresponding arguments. Not only will it insert your methods but it will create corresponding view files with it inside of your views directory.

This will definitely speed up your creation of the project with one caveat. You will need to change the routes file as it sets up custom routes too. This isnt bad if you are building your own rails app but will mess you up as it does not match the default rails routes that the assessment tests for.
```bash
bundle exec rails g controller CONTROLLERNAME METHOD1 METHOD2 METHOD3

## bundle exec rails g controller Users new edit show index create update destroy
```
### Level 3
Don't worry about which methods you add on the controller. Add all of them. Deleting is easier than creating and the tests dont test random paths.

## Set Up Your Bash Aliases
Tired of typing `bundle exec rails`? Set up an alias or many aliases. You can always change it after the assessment. Add this in your `~/.bash_profile` or wherever you have your aliases.
```bash
alias ber="bundle exec rails"
alias rr="bundle exec rails routes"
## And so much more :-D
```

## One Line Associations
Want to make your model associations on a single line rather than 3 or 4?  It is possible by naming the method of the association the name of the model you are associating it with.  There are many requirements to do this but with the amount of data/tables/models/classes we are dealing with, over 50% of the time it can be done.  here are the requirements

* examples are given with comments table with a foreign key of user_id associating it with the users table
### belongs_to
- the name of the method needs to be the name of the model it is association with in lower case.  You can also look at it as the singular form of the table it is associated with
(e.g. method can not be author.  It must be user)
- the table that the model belongs to must have the foreign key with the singular form of the table it is associating with with '\_id'
(e.g. the column on the comments table can not be author_id, it must be user_id)

### has_many
- the name of the method needs to be the plural form of the model it is associated with in lower case.  You can also look at it as the name of the table it is associated with.
(e.g. method can not be replies.  It must be comments)
- the table that it is associated with must have a foreign key with the the name of the model with '\_id'
(e.g. the column on the comments table can not be author_id, it must be user_id)

## Some Other Tricks

### Scaffolds
I really don't recommend these as it is like bringing a rocket launcher to a water gun fight but if you find out how to use it it is great as it is like running `bundle exec rails g model` AND `bundle exec rails g controller` in one.  It sets up the routes, forms in your views, and hooks up each route to function with your forms like a full blown app.  THE BAD THING:::: you're really going to need to modify a LOT in order to make it work with the assessment.  If you don't know any of the helper methods that rails gives in the views and controllers, you might be set back more than if you just stuck with the beginner level stuff.  I've been doing rails for almost month now and still have yet to get a scaffold to work in my favor.

If what I said didn't scare you, here is the command to create a scaffold
```
# notice it is the model name so it is singular and will pluralize where needed
bundle exec rails g scaffold MODELNAME COLUMNNAME1 COLUMNNAME2 COLUMNNAME3

## bundle exec rails g scaffold User username age:integer password_digest session_token
```

### Get Really Good At The Usual
You already know there are going to be users and forms.  Get really good at building creating the methods on the Users model and really good at making a fantastic form skeleton.  NO, I did not say form "partial".  Partials are only good if you are looking to maintain the code later or present your code base to someone.  We aren't going for code beauty here.  We are going for raw speed.  Know how to build a great form skeleton that you can *COPY* and *PASTE* for all of your other forms you'll need.  It should have your `form` elements, method and action attributes, text and submit inputs.  Below is my example that what I plan on doing is getting that single one done then pasting it in all of my edit and new views.

```ruby
<form method="<%= url %>" action="post">
  <input type="hidden" name="_method" action="<%= action %>">
  <label>
    Label
    <input type="text" name="<%= class %>[ column ]" value="">
  </label>
  <input type="submit" value="">
</form>
```

As you can see this is a very bare skeleton that I can easily modify with copy and pasting and deleting.  The great thing is I can easily modify this for a button in the case of making a DELETE request.

### Always Initiate Flash
The worst thing in the world is getting an error that says `Can't find _____ on nil class` (something along those lines).  Many times this happens when you forget that you have a element trying to display flash[:errors] (or notices) and flash is not defined).  There is a simple way around this.  Always have it defined!

This is the way I do it:
```ruby
class UsersController < ApplicationController
  before_action :initiate_flash, only: [:create, :update, :destroy]

  private

  def initiate_flash
    flash[:errors] = []
    flash[:notices] = []
  end
end
```

### Set Current User Instance Variable
Yes yes yes... you can set a helper method for current user but why when you are taking an assessment?  Just set the current user as an instance variable to save you the key strokes and to prevent the mind juggling.  Also this way you can see if they are logged in or logged out by as single instance variable.

Be mindful though.  The way the tests set the current user is though this method so make sure your naming is correct.

This is the way I do it:
```ruby
class ApplicationController

  def current_user
    User.find_by(id: session[:session_token]) if session && session[:session_token]
  end

  def set_current_user
    @current_user = current_user
  end
end

class UsersController < ApplicationController
  before_action :set_current_user

end

class SessionsController < ApplicationController
  before_action :set_current_user

end
```

### Disclaimer
Of course I'm not saying this is the most beautiful style of code but it is fast.  Pick and choose what helps you and if not, throw it away.  Best of luck to everyone!
