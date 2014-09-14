#Setting up Authentication with BCrypt and Rails

**Thanks to Kevin Ng for writing this great tutorial!**

In-Class Authentication Examples/Practice:

+ [Rails-Auth-Complete](https://github.com/ga-students/WDI_LA_9/tree/master/05-week/rails-auth-complete)

+ [Loggy-Inny Lab](https://github.com/ga-students/WDI_LA_9/tree/master/05-week/loggy-inny-lab)

Free online "Ruby On Rails Tutorial", chapters 6-8 [(also available on Amazon.com)](http://www.amazon.com/Ruby-Rails-Tutorial-Addison-Wesley-Professional/dp/0321832051/)

* [Chapter 6, User Model](http://draft.railstutorial.org/book/modeling_users)
* [Chapter 7, Sign Up](http://draft.railstutorial.org/book/sign_up)
* [Chapter 8, Log In/Log Out](http://draft.railstutorial.org/book/log_in_log_out#cha-log_in_log_out)


## 1. Setup Rails Environment
In Terminal - Create new app and cd into the new app directory

	 rails new ( app name ) --skip-active-record * creates new rails app directories 
 	 cd ( new app ) => cd into newly created directory



##2. Mongoid & Bcrypt
In editor - open Gemfile add the following gems

	gem 'mongoid', github: 'mongoid/mongoid' * Gets the latest from GitHub
	gem 'bcrypt', '~> 3.1.7'


##4. Do a Bundle Install in terminal
In Terminal - will be creating mongoid.yml file

	bundle install
	rails g mongoid:config 


##5. Run Mongod in terminal
In Terminal - runs mongoid

	mongod

##6. Run server in terminal 
 In Terminal - be sure to be in working directory(IMPORTANT)

	rails s


##7. Create user model
In Terminal - creating the rails model

	rails g model user

##8. Create users controller
In Terminal - creating the rails model

	rails g controller users new

##9. Configure routes.rb
In Editor

	delete get “users/new” (generated from previous step)
	add get ‘/signup’ => ‘users#new', as: :signup
	
	if you check your localhost:3000/signup in browser at this point you’ll see 	output below		
>	
##Users#new
######Find me in app/views/users/new.html.erb 



##10. User Model 
In Editor - app > model > user.rb  



	require 'bcrypt'
	
	class User
	  include Mongoid::Document
	
	  field :name, type: String
	  field :email, type: String
	  field :password_digest, type: String
	
	  attr_reader :password
	
	  def password=(new_password)
	    self.password_digest = BCrypt::Password.create(new_password)
	  end
	
	  def authenticate(test_password)
	    if test_password && BCrypt::Password.new(self.password_digest) == test_password
	      self
	    else
	      false
	    end
	  end
	
	end




##11. User Controller 
In Editor - app > controllers > users_controller.rb  

	class UsersController < ApplicationController  
	def new
	       @user = User.new
	  end
	
	  def create
	     @user = User.new(params.require(:user).permit(:name, :email, :password))
	     if @user.save
	          # log the user in
	          redirect_to root_path
	     else
	          render 'new'
	     end
	  end
	end


##12. Edit routes.rb to add the path to users 
In Editor - add to the routes.rb in config directory

	root 'users#new' - for now but will change to root ‘sessions#new’ once generated

  	get 'users/new' => 'users#new'
  	post 'users/' => 'users#create'


##13. Edit User new.html.erb 
In Editor - app > views > users > new.html.erb

	<h1>Sign Up!</h1>

	<%= form_for @user do |f| %>
	     <%= f.label :name %>
	     <%= f.text_field :name %>
	
	     <%= f.label :email %>
	     <%= f.text_field :email %>
	
	     <%= f.label :password %>
	     <%= f.text_field :password %>
	
	     <%= f.submit "Sign Me Up!" %>
	<% end %>


##14. Create sessions controller edit In Terminal
In Terminal 	
	
	rails g controller sessions new


##15. Edit sessions_controller.rb
In Editor - app > controllers > sessions > sessions_controller.rb

	class SessionsController < ApplicationController
	
		def new
		end
	
		def create
			begin
				user = User.find_by({email: params[:session][:email]})
			rescue
				flash[:error] = 'Email not found.'			
			end
	
			if user && user.authenticate(params[:session][:password])
				log_in(user)
				redirect_to root_path
			else
				flash[:error] ||= 'Try again.'
				render 'new'
			end
		end
	
		def destroy
			
			log_out
			redirect_to root_path
		end
	end



##16. Set routes.rb for sessions controller
  In Editor - now you can reset the root to sessions#index
  
  	root 'sessions#index'
	get '/login' => 'sessions#new', as: :sessions
 	post '/login' => 'sessions#create' 	
	delete '/logout' =>'sessions#destroy', as: :log_out

##17. Edit sessions_helper.rb file
In Editor - app > helpers > sessions_helper.rb

	module SessionsHelper
	
		################################
		# Ensure current_user returns 
		#    a User object from MongoDB
		################################
	
		# LOG IN: set user ID cookie in user's browser
		def log_in(user)
			cookies.permanent[:cookie_id] = user.id
			
			@current_user = user
		end
	
		# LOG OUT: remove cookie from user's browser
		def log_out
			cookies.delete(:cookie_id)
	
		end
	
		# true if user logged in
		def logged_in?
			cookies[:cookie_id] ? true : false
			
		end
	
	
		################################
		# Ensure current_user returns 
		#    a User object from MongoDB
		################################
	
		# If not already set, retrieve user from MongoDB
		def current_user
			if logged_in?
				@current_user ||= User.find(cookies[:cookie_id])
			else
				nil
			end
		end
	
		# current_user Setter (similar to attr_writer)
		def current_user=(user)
					@current_user = user
		end
	end

##18. Include session helper to application_controller 
In Editor - app > controllers > application_controller.rb
- add just before the end controller.

	include SessionsHelper


##19. Create a new new.html.erb for session directory in the views
In Editor - app > views > sessions 

 
	<h1>Sign In!</h1>
	
	<%= form_for(:session, url: sessions_path) do |f| %>
		<%= f.label :email %> <%= f.text_field :email %>
		<%= f.label :password %> <%= f.password_field :password %>
	
		<%= f.submit "Sign In!" %>
	<% end %>
	
	<%= link_to "New User!", signup_path %>
	

##20. Create a new index.html.erb for session directory in the views
In Editor - app > views > sessions 
	
	<p>This page will tell you if you are logged in or not!</p>
	
	<%= link_to "Sign up!", signup_path %><br>
	<%= link_to "Log in!", sessions_path %><br>
	<%= link_to "Log out!", log_out_path, method: :delete%>
