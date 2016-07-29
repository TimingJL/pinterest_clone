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
![image](https://github.com/TimingJL/pin_board/blob/master/pic/basic_new_page.jpeg)


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
![image](https://github.com/TimingJL/pin_board/blob/master/pic/first_pin.jpeg)

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
![image](https://github.com/TimingJL/pin_board/blob/master/pic/edit_link.jpeg)
![image](https://github.com/TimingJL/pin_board/blob/master/pic/edit_page.jpeg)

To be continued...