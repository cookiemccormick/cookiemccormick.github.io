---
layout: post
title:      "Dark and Stormy - Rails Project"
date:       2019-05-01 03:47:41 +0000
permalink:  dark_and_stormy_-_rails_project
---


My third Flatiron Project is a Rails application for creating cocktail recipes called Dark and Stormy.  A user is able to create cocktails, browse other user's cocktails and comment on cocktails.  

I love me some cocktails, but if you read my previous post, you know that I'm pregnant and can't have any, but one can dream for now.  

#### Model/Validation/Scope Requirements

The Rails Project required the following relationships:

[x] Include at least one has_many relationship<br>
[x] Include at least one belongs_to relationship<br> 
[x] Include at least two has_many through relationships<br> 
[x] Include at least one many-to-many relationship<br> 
[x] The "through" part of the has_many through includes at least one user submittable attribute, that is to say, some attribute other than its foreign keys that can be submitted by the app's user<br>
[x] Include reasonable validations for simple model objects (list of model objects with validations e.g. User, Recipe, Ingredient, Item)<br>
[x] Include a class level ActiveRecord scope method (model object & class method name and URL to see the working feature e.g. User.most_recipes URL: /users/most_recipes)

It really helped to draw out a diagram to better see and understand the relationships between the models.  Mine was done old-school on paper, but here is a nicer looking entity-relationship diagram created by the rails.erd gem.  

![](https://i.imgur.com/K0P2XS8.png)

The `User` model requires an `email` and is the unique identifier for users.  The `email` is saved downcased to ensure case-insensitive uniqueness.  The user model also requires a `name` and `password` for signup. When signing in using Facebook the password is not required.

```
class User < ApplicationRecord
  has_many :recipes, dependent: :destroy
  has_many :comments, dependent: :destroy

  validates :name, presence: true
  validates :email, presence: true, uniqueness: true
  validates :password, presence: true, unless: :uid?

  has_secure_password validations: false

  before_save :downcase_fields

  scope :alphabetical_name, -> { order("name asc") }

  def downcase_fields
    email.downcase!
  end
end
```

One of the trickiest parts in this application is creating a recipe.  A recipe must have a unique `name`,  `description`, `instructions` and at least one ingredient item with a `quantity` and `ingredient`.  The recipe form consists of a nested `quantity` and `ingredient` form with 10 blank fields.  The application then checks to see if there are any blank fields and removes them prior to saving.

```
class Recipe < ApplicationRecord
  belongs_to :user
  has_many :recipe_ingredients, dependent: :destroy
  has_many :ingredients, through: :recipe_ingredients
  has_many :comments, dependent: :destroy

  validates :name, presence: true, uniqueness: true
  validates :description, presence: true
  validates :instructions, presence: true
  validate :validate_recipe_ingredients

  accepts_nested_attributes_for :recipe_ingredients, :reject_if => proc {|attr| attr[:quantity].blank? && attr[:ingredient_attributes][:name].blank?}

  scope :most_recent, -> (limit) { order("created_at desc").limit(limit) }
  scope :alphabetical_name, -> { order("name asc") }

  def validate_recipe_ingredients
    errors.add(:recipe_ingredients, "must have at least one quantity and ingredient listed") if recipe_ingredients.length < 1
  end

  def build_empty_ingredients
    10.times { recipe_ingredients.build.build_ingredient }
  end
end
```

A user is able to comment on recipes.  The comments are created using a nested resource through recipes - `recipes/1/comments/new`. A comment is a pretty simple model and only requires a body string field for the comment content. They're associated to the user and the recipe automatically using the nested resource.

```
class Comment < ApplicationRecord
  belongs_to :user
  belongs_to :recipe

  validates :body, presence: true, allow_blank: false
  validates :recipe_id, presence: true
  validates :user_id, presence: true

  scope :most_recent, -> { order("created_at desc") }

  def self.comment_count
    if count == 0
      "This recipe has no comments."
    elsif count == 1
      "1 comment"
    else
      "#{count} comments"
    end
  end
end
```

`recipe_ingredients` is our join table between `Recipe` and `Ingredient`.  A user is able to add a `quantity` attribute. These rows are managed solely through the nested form on the new and edit recipe pages.

```
class RecipeIngredient < ApplicationRecord
  belongs_to :recipe
  belongs_to :ingredient

  validates :quantity, presence: true
  validates :ingredient, presence: true
  validates :recipe, presence: true
  validate :validate_ingredient

  def ingredient_attributes=(ingredient_attributes)
    ingredient_attributes.values.each do |attribute|
      self.ingredient = Ingredient.find_or_initialize_by(name: attribute)
    end
  end

  def validate_ingredient
    if ingredient.nil? || ingredient.name.blank?
      errors.add(:ingredient, "must list one ingredient")
    end
  end
end
```

The simplest model in the application is the `Ingredient` model. All ingredients created are unique and have a `name`. Ingredients are associated to their recipes using a has_many/through relationship and the `recipe_ingredients` join table.

```
class Ingredient < ApplicationRecord
  has_many :recipe_ingredients
  has_many :recipes, through: :recipe_ingredients

  validates :name, presence: true, uniqueness: true
end
```

#### Signup/Login/Logout

A `User` can be created through the signup form or can choose to login through Facebook.  The `bcrypt` gem and the `has_secure_password` method are used for hashing the passwords.

/signup

```
<h1>Signup</h1>

<%= form_for @user, url: signup_path do |f| %>

  <% if @user.errors.any? %>
    <h2>There were some errors: </h2>
    <ul>
      <% @user.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
    </ul>
  <% end %><br>

  Username:<%= f.text_field :name %><br>
  Email:<%= f.email_field :email %><br>
  Password:<%= f.password_field :password %><br>
  Password Confirmation:<%= f.password_field :password_confirmation %><br><br>

  <%= f.submit "Signup" %>
<% end %><br>

<div><%= link_to "Welcome to Dark and Stormy", root_path %></div>
```

/login

```
<h1>Login</h1>

<%= form_for :session, url: login_path do |f| %>

  Email:<%= f.email_field :email %>
  Password:<%= f.password_field :password %>
  <%= f.submit "Login" %>
<% end %><br>

<%= link_to 'Login with Facebook!', '/auth/facebook' %><br>


<%= link_to "Signup", signup_path %>
```

/logout

```
<div>
  <% if current_user %>
    <%= form_for current_user, url: logout_path, method: :post do |f| %>
      <%= f.submit "Logout" %>
    <% end %>
  <% end %>
<div>
```

The /logout form is listed in layouts to keep the application DRY.  A partial for links is also listed in the layouts.  Creating a recipe and editing a recipe also renders a partial.

#### Nested Resources

Another requirement to the project is to created nested resource pages for new and index/show.

```
Rails.application.routes.draw do
  get '/signup', to: 'users#new'
  post '/signup', to: 'users#create'

  get '/login', to: 'sessions#new'
  post '/login', to: 'sessions#create'
  post '/logout', to: 'sessions#destroy'

  get '/home', to: 'welcome#home'

  resources :recipes
  resources :users

  resources :users, only: [:show] do
    resources :recipes, only: [:show, :index]
  end

  resources :recipes, only: [:show, :index] do
    resources :comments, only: [:index, :new, :create]
  end

  get '/auth/facebook/callback', to: 'sessions#create'

  root 'static#home'
end
```

![](https://i.imgur.com/7aBykYR.png?1)<br><br>
![](https://i.imgur.com/XyaUlNG.png)

#### Future Features

Possible ideas:

* Images of cocktails uploaded by the user
* Ratings
* CSS styling
* Finding recipes by ingredients

**Application Link:**

https://github.com/cookiemccormick/dark-and-stormy


















