# Ruby on Rails Tutorial 1
## https://www.youtube.com/watch?v=fmyvWz5TUWg
### I had done this, but did not finish due to the understanding is what I need. I do not need a demo site to my port.
> Install with these commands on WSL2 or a linux host.
```
sudo apt update && sudo apt upgrade -y
sudo apt install gnupg2
gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB

curl -sSL https://get.rvm.io | bash -s stable
source /home/xander/.rvm/scripts/rvm

rvm requirements
rvm list known
rvm install ruby
gem install rails

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
command -v nvm
bash
nvm install node
nvm list

npm install -g npm@latest
npm install -g yarn
```

> Start the rails server
Choose your directory that you wish to start building in.
```
rails new .
rails s

# What is ruby on rails?

> It is an MVC Architecture (Models, View, Controller) 

Models are databases. 

Views are web pages.

Controllers are the brains behind the scenes. Think about djangos' view.py files.
```

# Creating a controller and start page
> The below command will make a new web page and folder in app/views/home/
```
rails g controller home index
```

# Linking and refrencing
> Basically all you need to know is that 'a' tags are useless, use...
```
# For rendering assets e.g.
<%= render 'home/header' %>

# For linking whole pages e.g.
<%= link_to 'About Us', {dir}_{filename}_path, class:"nav-link" %>
```

# Starting a database schema (scaffold)
> This is building a CRUD (Create, Read, Update, Destroy) using your database of choice. You have to specify what you are using in the gemfile. By default we use sqlite3 
```
rails g scaffold friends first_name:string last_name:string email:string phone:string twitter:string
rails db:migrate
```

# Using Gems (3rd Party Things that just WORK!)
> We will utilize one for flexible authentication using Warden. It is called <a href="https://rubygems.org/gems/devise">devise!</a>
1. Place it in your Gemfile ```gem 'devise', '~> 4.8', '>= 4.8.1'```
2. understand what components require additional information through their github or webpage.

# Associations
> We can associate two tables to perform certain functions between databases. For example, we want specific users who have signed up using devise to have their own friendlist.
1. Inside of app/models we have our associations based on the two databases that were created through scaffold (friend, user)
2. friend is currently empty, while user is set up through devise to have specific assocations for certain user states.
3. To add an association for the friend database, we have to add a new line in friend.rb ```belongs_to :user```. This points to the user database as an entity under many users.
4. Therefore, we must add a new line for ```has_many :friends``` in user.rb.
5. Now that we have added the association, we must now make sure the database has record of which user belongs to which friend list. 
6. We will create a new database schema / migration for the friends schema to associate. ```rails g migration add_user_id_to_friends user_id:integer:index```. We add the :index to provide searching and making things quicker based on ID. Don't forget to ```db:migrate```
7. You can see what the current_user's id is by utilizing devise ```<% = current_user.id %>```. This is unique and a value that is already assigned. You can show this on the user's profile, or whatever you wish. 
8. We can place this current_user id value in the friend's table by doing... ```<%= form.number_field :user_id, id: :friend_user_id, class:"form-control", value: current_user.id, type: :hidden %>```. This will submit the form to the database, and with the ```:hidden``` type, it now will not show on the page that it is "editable".
9. The user ID is not being passed to the database yet because there are too many form fields (parameters) being sent to the DB through the friend creation page. You would edit this through the app/controllers.
10. Editing the existing CRUD controller file for friends by adding a new :user_id permit parameter ```params.require(:friend).permit(:first_name, :last_name, :email, :phone, :twitter, :user_id)```. We still have to limit other friends from unassociated users. To do this, we will use before actions

# Before Actions
1. Now we must prevent unauthenticated users access editing friends lists to other users. To do this, we must take steps before performing any action. Inside the app/controllers for friends, you can do ```before_action``` at the top of friends_controller.rb.
2. ```before_action :authenticate_user!, except: [:index, show]```. The index and show exceptions correspond to the rest of the method definitions in the controller rb file. 
3. We still need to limit which friends are associated to which users, to do this, we will create a new definition in the controller file:

```
before_action: correct_user, only: [:edit, :update, :destroy]

...
... 

def correct_user
  @friend = current_user.friends.find_by(id: params[:id])
  redirect_to friends_path, notice: "Not Authorized to Edit This Friend" if @friend.nil?
end # search for friends that have been created / edit before by params[:id], if it has, it is not nil. But if it is nil, show the notice: "not authorized...." 
```

Now we have to edit the way the function for new, editing and destroying works in the controller. It has to go through an association of the current user and their build. 
```
def new
  #@friend = Friend.New    # this is the original line
  @friend = current_user.friends.build
end 

...
...
def create
  #@friend = Friend.new(friend_params) # this is the original line
  @friend = current_user.friends.build(friend_params)
...
end
```
4. Now we have to limit through ruby logic on the friends list of other users by editing the views/friends/index.html.erb file.
``` 
<% if friend.user == current_user %>
{ELEMENTS THAT SHOW EACH USER}
<% end %>
```