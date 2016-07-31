# Learning by Doing 

![Ubuntu version](https://img.shields.io/badge/Ubuntu-16.04%20LTS-orange.svg)
![Rails version](https://img.shields.io/badge/Rails-v5.0.0-blue.svg)
![Ruby version](https://img.shields.io/badge/Ruby-v2.3.1p112-red.svg)

Mackenzie Child's video really inspired me. So I decided to follow all of his rails video tutorial to learn how to build a web app. Through the video, I would try to build the web app by my self and record the courses step by step in text to facilitate the review.


# Project 4: How To Build A Pinterest Clone With Rails

This time we're going to build a Pinterest like application which includes the Pins, Users, Image uploading, and Voting as well. A user must sign-in in order to  submit a new pin or vote, as well as to delete a pin. And then we got some amazingly going on, so the Pins will scale to the size of the PinBoard(window size).

https://mackenziechild.me/12-in-12/4/


### Highlights of this course
1. Users
2. Pins
3. Image Uploading
4. Voting
5. HAML
6. Masonry
7. Bootstrap 


# Create a PinBoard
```console
$ rails new pin_board
```

Chage directory to the pin_board. Under `pin_board/Gemfile`, add `gem 'therubyracer'`, save and run `bundle install`.      

Note: 
Because there is no Javascript interpreter for Rails on Ubuntu Operation System, we have to install `Node.js` or `therubyracer` to get the Javascript interpreter.

```console
$ bundle install
```

Then run the `rails server` and go to `http://localhost:3000` to make sure everything is correct.

Then we're going to install a few gems that we're going to use in this project.
```
	gem 'haml', '~>4.0.5'
	gem 'bootstrap-sass', '~> 3.2.0.2'
	gem 'simple_form', github: 'kesha-antonov/simple_form', branch: 'rails-5-0'
	gem 'devise'
```

Let's do bundle install and restart the server.

Then, we're going to generate our model for our Pins.
```console
$ rails g model Pin title:string description:text
$ rake db:migrate
```

And let's go ahead to generate our controller.
```console
$ rails g controller Pins
```
Then we're going to setup a few pages. First thing we'll do index to hold the Pins.
In `app/controllers/pins_controller.rb`
```ruby
class PinsController < ApplicationController
	def index
	end
end
```

And then in our `config/routes.rb`
```ruby
Rails.application.routes.draw do
  resources :pins

  root "pins#index"
end
```

Then let's make up template under `app/views/pins`, and create a new file `index.html.haml`.         
In `app/views/pins/index.html.haml`
```haml
%h1 This is the index placeholder.
```

# Basic CRUD

Let's add the ability of CRUD(the Create, Read ,Update , and Destroy)     
In `app/controllers/pins_controller.rb`
```ruby
class PinsController < ApplicationController
	def index
	end

	def new
		@pin = Pin.new
	end

	def create
		@pin = Pin.new(pin_params)
	end

	private

	def pin_params
		params.require(:pin).permit(:title, :description)
	end
end
```

### Create
Next, we need to create a new file `new.html.haml` for our new form.        
So in `app/views/pins/new.html.haml`
```haml
%h1 New Form

= render 'form'

= link_to "Back", root_path
```

Let's create another file `_form.html.haml` under `app/views/pins`.      
To use this on `simple_form`, we're gonna do
```console
$ rails g simple_form:install --bootstrap
```

Go back to our `app/views/pins/_form.html.haml`
```haml
= simple_form_for @pin, html: { multipart: true } do |f|
	- if @pin.errors.any?
		#errors
			%h2
				= pluralize(@pin.error.count, "error")
				prevented this Pin form saving
			%ul
				-@pin.errors.full_messages.each do |msg|
					%li = msg

	.form-group
		= f.input :title, input_html: { class: 'form-control'}

	.form-group
		= f.input :description, input_html: { class: 'form-control'}

	= f.button :submit, class: "btn btn-primary"
```
Note:     
`pluralize` is a rails helper. What the pluralize does is anything past account of one, so at 2,3 on , it will automatically pluralize it for us.       
http://rails.ruby.tw/getting_started.html        
http://apidock.com/rails/ActionView/Helpers/TextHelper/pluralize

Back to the browser, go to `http://localhost:3000/pins/new`
![image](https://github.com/TimingJL/pinterest_clone/blob/master/pic/basic_new_page.jpeg)


Then, let's go back to our `app/controllers/pins_controller.rb`
```ruby
class PinsController < ApplicationController
	...
	...

	def create
		@pin = Pin.new(pin_params)

		if @pin.save
			redirect_to @pin, notice: "Successfully created new Pin"
		else
			render 'new'
		end
	end

	...
	...
```

### Show
And in `app/views/layouts`, first thing we wonna do is rename `application.html.erb` to `application.html.haml`.
Then convert the html tag to haml and add the flash message.
```haml
!!! 5
%html
%head
	%title PinBoard
	= csrf_meta_tags
	= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload'
	= javascript_include_tag 'application', 'data-turbolinks-track': 'reload'

%body
	- flash.each do |name, msg|
		= content_tag :div, msg, class: "alert alert-info"
			
	= yield 
```

To show the Pin, we have to back to the controller.      
In `app/controllers/pins_controller.rb`
```ruby
class PinsController < ApplicationController
	before_action :find_pin, only: [:show, :edit, :update, :destroy]
	
	def index
	end

	def show
	end

	def new
		@pin = Pin.new
	end

	def create
		@pin = Pin.new(pin_params)

		if @pin.save
			redirect_to @pin, notice: "Successfully created new Pin"
		else
			render 'new'
		end
	end

	private

	def pin_params
		params.require(:pin).permit(:title, :description)
	end

	def find_pin
		@pin = Pin.find(params[:id])
	end
end
```

Then we next need to create a view `show.html.haml` for show.      
In `app/views/pins/show.html.haml`     
```haml
%h1= @pin.title
%p= simple_format @pin.description

= link_to "Back", root_path
```

Now, we can create our first Pin and show it up.
![image](https://github.com/TimingJL/pinterest_clone/blob/master/pic/first_pin.jpeg)

So far, we have the flash message, title and description.


Then, on the `index.html.haml`, let's list out all the Pins.       
In `app/controllers/pins_controller.rb` 
```ruby
	...

	def index
		@pins = Pin.all.order("created_at DESC")
	end

	...
```


In `app/views/pins/index.html.haml` 
```haml
- @pins.each do |pin|
	%h2= link_to pin.title, pin
```

### Update
Now, we need to add the ability to `edit` and `destroy` a Pin as well.       
In `app/controllers/pins_controller.rb`
```ruby
class PinsController < ApplicationController
	before_action :find_pin, only: [:show, :edit, :update, :destroy]
	...
	...

	def edit
	end

	def update
		if @pin.update(pin_params)
			redirect_to @pin, notice: "Pin was Successfully updated!"
		else
			render 'edit'
		end
	end

	def destroy
	end
	...
	...
```
Next, we need to create a new file `edit.html.haml` for the edit page     
In `app/views/pins/edit.html.haml` 
```haml
%h1 Edit Pin

= render 'form'

= link_to 'Cancel', pin_path
```

So on the show page, let's add a link to the edit form.      
In `app/views/pins/show.html.haml`
```haml
%h1= @pin.title
%p= simple_format @pin.description

= link_to "Back", root_path
= link_to "Edit", edit_pin_path
```
![image](https://github.com/TimingJL/pinterest_clone/blob/master/pic/edit_link.jpeg)
![image](https://github.com/TimingJL/pinterest_clone/blob/master/pic/edit_page.jpeg)

### Destroy
Now, let's add the ability to destroy. So in our controller. let's do:     
`app/controllers/pins_controller.rb`
```ruby
	...
	...
	def destroy
		@pin.destroy
		redirect_to root_path
	end
	...
	...
```

Then, in our show, we need to add a `destroy link`.       
In `app/views/pins/show.html.haml`
```haml
%h1= @pin.title
%p= simple_format @pin.description

= link_to "Back", root_path
= link_to "Edit", edit_pin_path
= link_to "Delete", pin_path, method: :delete, data: {confirm: "Are you sure?"}
```

And we want to add a `New Pin` link in our `index.html.haml` page.     
``app/views/pins/index.html.haml``
```haml
= link_to "New Pin", new_pin_path
- @pins.each do |pin|
	%h2= link_to pin.title, pin
```


# Add Users
We're gonna to use `devise` gem.

```console
$ rails g devise:install
```

```
	Some setup you must do manually if you haven't yet:

	  1. Ensure you have defined default url options in your environments files. Here
	     is an example of default_url_options appropriate for a development environment
	     in config/environments/development.rb:

	       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

	     In production, :host should be set to the actual host of your application.

	  2. Ensure you have defined root_url to *something* in your config/routes.rb.
	     For example:

	       root to: "home#index"

	  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
	     For example:

	       <p class="notice"><%= notice %></p>
	       <p class="alert"><%= alert %></p>

	  4. You can copy Devise views (for customization) to your app by running:

	       rails g devise:views
```

in config/environments/development.rb:
```
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

And
```console
$ rails g devise:views
```

Next step is generate a devise model.
```console
$ rails g devise User
$ rake db:migrate
```

Let's restart the server and go to `http://localhost:3000/users/sign_up` to make sure everything is correct.
![image](https://github.com/TimingJL/pinterest_clone/blob/master/pic/basic_signup_page.jpeg)


So what we need to do next is make sure that each Pin that is created has a user assigned to it.
In `app/models/user.rb`
```ruby
class User < ApplicationRecord
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable
  has_many :pins
end
```

And in `app/models/pin.rb`
```ruby
class Pin < ApplicationRecord
	belongs_to :user
end
```

Then we need to generate migration so that our Pin has a user_id column.
```console
$ rails g migration add_user_id_to_pins user_id:integer:index
$ rake db:migrate
```

That's pop into the rails console
```console
$ rails c
```

In rails console
```console
> @pin = Pin.first
```
```
irb(main):001:0> @pin = Pin.first       
  Pin Load (0.5ms)  SELECT  "pins".* FROM "pins" ORDER BY "pins"."id" ASC LIMIT ?  [["LIMIT", 1]]       
=> #<Pin id: 2, title: "Pinterest", description: "Pinterest is a web and mobile application company ...", created_at: "2016-07-29 08:26:03", updated_at: "2016-07-29 08:26:03", user_id: nil>
```

You can see the `user_id: `nil`. So what I'll do is:
```console
> @user = User.first
> @pin.user = @user
> @pin
```

```
irb(main):001:0> @pin = Pin.first     
  Pin Load (0.5ms)  SELECT  "pins".* FROM "pins" ORDER BY "pins"."id" ASC LIMIT ?  [["LIMIT", 1]]
=> #<Pin id: 2, title: "Pinterest", description: "Pinterest is a web and mobile application company ...",       
created_at: "2016-07-29 08:26:03", updated_at: "2016-07-29 08:26:03", user_id: nil>
```
You can see the `user_id: 1`. Then
```console
> @pin.save
```

Let's test in `app/views/pins/show.html.haml`
```haml
%h1= @pin.title
%p= simple_format @pin.description
%p
Submitted by
= @pin.user.email
%br

= link_to "Back", root_path
= link_to "Edit", edit_pin_path
= link_to "Delete", pin_path, method: :delete, data: {confirm: "Are you sure?"}
```
![image](https://github.com/TimingJL/pinterest_clone/blob/master/pic/user_test.jpeg)

If we create a new Pin right now, it's not going to save because we didn't have `@pin.user.email` for that Pin.

So back to our `app/controllers/pins_controller.rb`, we need to tweak the `new action` and `create action` a bit.       
We change
```ruby
def new
	@pin = Pin.new
end

def create
	@pin = Pin.new(pin_params)

	if @pin.save
		redirect_to @pin, notice: "Successfully created new Pin"
	else
		render 'new'
	end
end	
```
to
```ruby
def new
	@pin = current_user.pins.build
end

def create
	@pin = current_user.pins.build(pin_params)

	if @pin.save
		redirect_to @pin, notice: "Successfully created new Pin"
	else
		render 'new'
	end
end
```

# Styling and Bootstrap
https://github.com/twbs/bootstrap-sass       

### Import Bootstrap
We already have the gem install.      
Next, we need to go into `app/assets/stylesheets` and rename `application.css` to `application.css.scss`.       
And then, we gonna to import Bootstrap styles in `app/assets/stylesheets/application.css.scss`:

	@import "bootstrap-sprockets";
	@import "bootstrap";

![image](https://github.com/TimingJL/pinterest_clone/blob/master/pic/bootstrap_import.jpeg)

Require Bootstrap Javascripts in `app/assets/javascripts/application.js`:
```js
//= require jquery
//= require bootstrap-sprockets
```

### Add Navbar
So to start, I'm going to add the application layout file.        
Under `app/views/layouts/application.html.haml`
```haml
!!! 5
%html
%head
	%title Pin Board
	= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true
	= javascript_include_tag 'application', 'data-turbolinks-track' => true
	= csrf_meta_tags

%body
	%nav.navbar.navbar-default
		.container
			.navbar-brand= link_to "Pin Board", root_path

			- if user_signed_in?
				%ul.nav.navbar-nav.navbar-right
					%li= link_to "New Pin", new_pin_path
					%li= link_to "Account", edit_user_registration_path
					%li= link_to "Sign Out", destroy_user_session_path, method: :delete
			- else
				%ul.nav.navbar-nav.navbar-right
					%li= link_to "Sign Up", new_user_registration_path
					%li= link_to "Sign In", new_user_session_path
	.container
		- flash.each do |name, msg|
			= content_tag :div, msg, class: "alert alert-info"
		= yield
```
![image](https://github.com/TimingJL/pinterest_clone/blob/master/pic/navbar.jpeg)


### Add Wrapper
Let's add wrapper around the new and edit page.     
Under `app/views/pins/new.html.haml`
```haml
.col-md-8.col-md-offset-3
	%h1 New Form
	= render 'form'
	= link_to "Back", root_path
```

Under `app/views/pins/edit.html.haml`
```haml
.col-md-8.col-md-offset-3
	%h1 Edit Pin
	= render 'form'
	= link_to 'Cancel', pin_path
```

# Image Uploading
Next, we want to add the ability to upload images.
So we add `paperclip` to our `Gemfile`.          
```
gem 'paperclip', '~> 4.2.0'
```
https://github.com/thoughtbot/paperclip      

Then we run `bundle install` and restart our server.      
Now, we need to add `has_attached_file` and the `validates_attachment_content_type` to      
`app/models/pin.rb`
```ruby
class Pin < ApplicationRecord
	belongs_to :user

	has_attached_file :image, :styles => { :medium => "300x300>" }
	validates_attachment_content_type :image, :content_type => /\Aimage\/.*\Z/
end
```

And next, we need to add migration. We can do that by running:
```console
$ rails g paperclip pin image
$ rake db:migrate
```

Next, in our form `app/views/_form.html.haml`
```haml
= simple_form_for @pin, html: { multipart: true } do |f|
	- if @pin.errors.any?
		#errors
			%h2
				= pluralize(@pin.error.count, "error")
				prevented this Pin form saving
			%ul
				-@pin.errors.full_messages.each do |msg|
					%li = msg

	.form-group
		= f.input :image, input_html: { class: 'form-control'}

	.form-group
		= f.input :title, input_html: { class: 'form-control'}

	.form-group
		= f.input :description, input_html: { class: 'form-control'}

	= f.button :submit, class: "btn btn-primary"
```

Now, let's do `New Pin` in browser:
![image](https://github.com/TimingJL/pinterest_clone/blob/master/pic/image_upload_new.jpeg)

One last thing we need to do is add `image` to our `pin_params` action.
`app/controllers/pins_controller.rb`
```ruby
def pin_params
	params.require(:pin).permit(:title, :description, :image)
end
```

And let's go into the `app/views/show.html.haml`, and add `image_tag` to the top.
```haml
= image_tag @pin.image.url(:medium)
%h1= @pin.title
%p= simple_format @pin.description
%p
Submitted by
= @pin.user.email
%br

= link_to "Back", root_path
= link_to "Edit", edit_pin_path
= link_to "Delete", pin_path, method: :delete, data: {confirm: "Are you sure?"}
```
![image](https://github.com/TimingJL/pinterest_clone/blob/master/pic/image_upload_test.jpeg)

Now, let's add this to the `index.html.haml` as well.     
In `app/views/index.html.haml`
```haml
- @pins.each do |pin|
	= link_to (image_tag pin.image.url(:medium)), pin
	%h2= link_to pin.title, pin
```
![image](https://github.com/TimingJL/pinterest_clone/blob/master/pic/index_image.jpeg)


Then, I want to show the user which image they are editing or which image they are currently uploading.     
In `app/views/pins/edit.html.haml`
```haml
.col-md-8.col-md-offset-3
	%h1 Edit Pin
	= image_tag @pin.image.url(:medium)
	= render 'form'
	= link_to 'Cancel', pin_path
``` 


# Masonry
Masonry is a light-weight layout framework which wraps AutoLayout with a nicer syntax.        
https://github.com/kristianmandrup/masonry-rails      

Let's go to our `Gemfile`, we need a gem called `masonry-rails`.          
In `Gemfile`, we add this line, run bundle install and restart the server.
```
gem 'masonry-rails', '~> 0.2.1'
```

In `app/assets/javascripts/application.js`, under jquery
```js
//= require jquery
//= require jquery_ujs
//= require masonry/jquery.masonry
//= require bootstrap-sprockets
//= require turbolinks
//= require_tree .
```

To get this work, I'm going to add some styling and coffescript.
In `app/assets/javascripts/pin.coffee`
```coffee
$ ->
  $('#pins').imagesLoaded ->
    $('#pins').masonry
      itemSelector: '.box'
      isFitWidth: true
```


In our `app/views/pins/index.html.haml`, we need to add:
```haml
#pins.transitions-enabled
	- @pins.each do |pin|
		.box.panel.panel-default
			= link_to (image_tag pin.image.url), pin
			.panel-body
				%h2= link_to pin.title, pin
```


### Basic Styling
Then, let's add some styles (`*= require 'masonry/transitions'` and some css).
In `app/assets/stylesheets/application.css.scss`
```scss
/*
 * This is a manifest file that'll be compiled into application.css, which will include all the files
 * listed below.
 *
 * Any CSS and SCSS file within this directory, lib/assets/stylesheets, vendor/assets/stylesheets,
 * or vendor/assets/stylesheets of plugins, if any, can be referenced here using a relative path.
 *
 * You're free to add application-wide styles to this file and they'll appear at the bottom of the
 * compiled file so the styles you add here take precedence over styles defined in any styles
 * defined in the other CSS/SCSS files in this directory. It is generally better to create a new
 * file per style scope.
 *
 *= require 'masonry/transitions'
 *= require_tree .
 *= require_self
 */

@import "bootstrap-sprockets";
@import "bootstrap";

body {
	background: #E9E9E9;
}

h1, h2, h3, h4, h5, h6 {
	font-weight: 100;
}

nav {
	box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.22);
	.navbar-brand {
		a {
			color: #BD1E23;
			font-weight: bold;
			&:hover {
				text-decoration: none;
			}
		}
	}
}

#pins {
  margin: 0 auto;
  width: 100%;
  .box {
	  margin: 10px;
	  width: 350px;
	  box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.22);
	  border-radius: 7px;
	  text-align: center;
	  img {
	  	max-width: 100%;
	  	height: auto;
	  }
	  h2 {
	  	font-size: 22px;
	  	margin: 0;
	  	padding: 25px 10px;
	  	a {
				color: #474747;
	  	}
	  }
	  .user {
	  	font-size: 12px;
	  	border-top: 1px solid #EAEAEA;
			padding: 15px;
			margin: 0;
	  }
	}
}

#edit_page {
	.current_image {
		img {
			display: block;
			margin: 20px 0;
		}
	}
}

#pin_show {
	.panel-heading {
		padding: 0;
	}
	.pin_image {
		img {
			max-width: 100%;
			width: 100%;
			display: block;
			margin: 0 auto;
		}
	}
	.panel-body {
		padding: 35px;
		h1 {
			margin: 0 0 10px 0;
		}
		.description {
			color: #868686;
			line-height: 1.75;
			margin: 0;
		}
	}
	.panel-footer {
		padding: 20px 35px;
		p {
			margin: 0;
		}
		.user {
			padding-top: 8px;
		}
	}
}

textarea {
	min-height: 250px;
}
```
![image](https://github.com/TimingJL/pinterest_clone/blob/master/pic/index_page_with_Masonry.jpeg)


Then, let's styling the show page.      
In `app/views/pins/show.html.haml`
```haml
#pin_show.row
	.col-md-8.col-md-offset-2
		.panel.panel-default
			.panel-heading.pin_image
				= image_tag @pin.image.url
			.panel-body
				%h1= @pin.title
				%p.description= @pin.description
			.panel-footer
				.row
					.col-md-6
						%p.user
							Submitted by
							= @pin.user.email
					.col-md-6
						.btn-group.pull-right
							= link_to "Edit", edit_pin_path, class: "btn btn-default"
							= link_to "Delete", pin_path, method: :delete, data: { confirm: "Are you sure?" }, class: "btn btn-default"
```
![image](https://github.com/TimingJL/pinterest_clone/blob/master/pic/show_page_styling.jpeg)



Last, I wanna add a user under the title.         
In `app/views/pins/index.html.haml`
```haml
#pins.transitions-enabled
	- @pins.each do |pin|
		.box.panel.panel-default
			= link_to (image_tag pin.image.url), pin
			.panel-body
				%h2= link_to pin.title, pin
				%p.user
				Submitted by
				= pin.user.email
```
![image](https://github.com/TimingJL/pinterest_clone/blob/master/pic/add_user_in_index.jpeg)



# Voting
To do voting, we're going to need to use `acts_as_votable` gem.      
https://github.com/ryanto/acts_as_votable    

```
gem 'acts_as_votable', '~> 0.10.0'
```
Do bundel install and restar our server as well.
Next we need to do is migration.
```console
$ rails g acts_as_votable:migration
$ rake db:migrate
```

Then, inside of our pin model `app/models/pin.rb`, we add `acts_as_votable` on the top.
```ruby
class Pin < ApplicationRecord
	acts_as_votable
	belongs_to :user

	has_attached_file :image, :styles => { :medium => "300x300>" }
	validates_attachment_content_type :image, :content_type => /\Aimage\/.*\Z/
end
```

The next we need to add some routes.
In `config/routes.rb`
```ruby
Rails.application.routes.draw do
  devise_for :users
  resources :pins do
  	member do
  		put "like", to: "pins#upvote"
  	end
  end

  root "pins#index"
end
```

Next, inside of our `app/controllers/pins_controller.rb`, I'm going to create a `upvote action`.
```ruby
class PinsController < ApplicationController
	before_action :find_pin, only: [:show, :edit, :update, :destroy, :upvote]
	...
	...
	def upvote
		@pin.upvote_by current_user
		redirect_to :back
	end
	...
	...
```

Then, in our show page `app/views/pins/show.html.haml`
```haml
#pin_show.row
	.col-md-8.col-md-offset-2
		.panel.panel-default
			.panel-heading.pin_image
				= image_tag @pin.image.url
			.panel-body
				%h1= @pin.title
				%p.description= @pin.description
			.panel-footer
				.row
					.col-md-6
						%p.user
							Submitted by
							= @pin.user.email
					.col-md-6
						.btn-group.pull-right
							= link_to like_pin_path(@pin), method: :put, class: "btn btn-default" do
								%span.glyphicon.glyphicon-heart
								= @pin.get_upvotes.size
							= link_to "Edit", edit_pin_path, class: "btn btn-default"
							= link_to "Delete", pin_path, method: :delete, data: { confirm: "Are you sure?" }, class: "btn btn-default"
```
![image](https://github.com/TimingJL/pinterest_clone/blob/master/pic/voting.jpeg)






To be continued...