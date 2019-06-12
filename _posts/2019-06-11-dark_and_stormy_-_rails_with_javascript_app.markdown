---
layout: post
title:      "Dark and Stormy - Rails with JavaScript App"
date:       2019-06-11 22:41:36 -0400
permalink:  dark_and_stormy_-_rails_with_javascript_app
---


For Flatiron School's fourth project, we were asked to expand on our Rails app by incorporating JavaScript and a JSON API to add dynamic features.  

Dark and Stormy is a rails app for creating cocktail recipes.  The first step I made was to create a [duplicate of my repository](https://help.github.com/en/articles/duplicating-a-repository).   I wanted to keep my rails app intact and have a separate app just for this project.  Files listed under `.gitignore` are not copied over, so I needed to make a new `.env` file to hold my `omniauth-facebook` key and secret.  I also went ahead and created a new app on the [Facebook developer site](https://developers.facebook.com/).

## There are 5 main project requirements:

### 1. Must translate JSON responses from your Rails app into JavaScript Model Objects using either ES6 class or constructor syntax. The Model Objects must have at least one method on the prototype. (Formatters work really well for this.)

In order to create JSON responses, I added the following gems:
```
gem 'jquery-rails'
gem 'active_model_serializers'
```

JavaScript Model objects were created using ES6 classes.  I created a prototype function on the recipe model object for displaying the amount of comments a recipe has, if any exists.
```
class Comment {
  constructor(comment) {
    this.id = comment.id;
    this.body = comment.body;
    this.commenter = comment.commenter;
    this.createdAt = new Date(comment.created_at);
  }
}

class Ingredient {
  constructor(ingredient) {
    this.quantity = ingredient.quantity;
    this.name = ingredient.name;
  }
}

class Recipe {
  constructor(recipe) {
    this.id = recipe.id;
    this.name = recipe.name;
    this.description = recipe.description;
    this.instructions = recipe.instructions;
    this.createdAt = new Date(recipe.created_at);
    this.user = new User(recipe.user);
    this.comments = recipe.comments.map(json => new Comment(json));
    this.ingredients = recipe.ingredients.map(json => new Ingredient(json));
  }

  commentCountLabel() {
    let commentCount = null;

    if (this.comments.length === 0) {
      commentCount = "There are no comments.";
    } else if (this.comments.length === 1) {
      commentCount = "1 comment";
    } else {
      commentCount = `${this.comments.length} comments`;
    }
    return commentCount;
  }
}

class User {
  constructor(user) {
    this.id = user.id;
    this.name = user.name;
  }
}
```

### 2. Must render at least one index page (index resource - 'list of things') via JavaScript and an Active Model Serialization JSON Backend.

When a user logs in, the first page view is the homepage which lists the 5 most recently created recipes.  This index view along with the recipes index view is fetched via AJAX GET requests on load.  

All the JavaScript code for fetching and posting is in `main.js`.  The recipe object has many nested attributes, having all the code in one spot similifies this project's requirements.  I may separate the code into different files later when I expand on the application.  Here is a look at the recipes index controller, JS and views:

RECIPES CONTROLLER
```
def index
  if params[:user_id]
    @recipes = User.find(params[:user_id]).recipes.alphabetical_name
  else
    @recipes = Recipe.alphabetical_name
  end
  respond_to do |format|
    format.html { render :index }
    format.json { render json: @recipes }
  end
end
```

RECIPE JS (main.js)
```
function getAllRecipes() {
  if ($("#recipes").length === 0) {
    return;
  }

  const userId = $("#recipes").attr("data-user-id");
  const url = userId ? `/users/${userId}/recipes.json` : '/recipes.json';

  $.get(url).done((data) => {
    const recipes = data.map(json => new Recipe(json));

    if (recipes.length === 0) {
      $("#recipes").html("<p>There are currently no recipes.</p>");
      return;
    }

    const recipeList = recipes.map((recipe) => {
      return `
        <li>
          <a href="/recipes/${recipe.id}">${recipe.name}</a>
          created ${recipe.createdAt.toLocaleDateString()}<br>
          ${recipe.description}<br>
          by <a href="/users/${recipe.user.id}">${recipe.user.name}</a><br><br>
        </li>
      `;
    })

    $("#recipes").html(`<ul>${recipeList.join('')}</ul>`);
  });
}

$(getAllRecipes);
```

INDEX.HTML.ERB
```
<h1>Recipes</h1>

<div id="recipes" data-user-id="<%= params[:user_id] %>"></div>

<%= render 'shared/links' %><br><br>
```

RECIPES INDEX VIEW
![](https://i.imgur.com/Tjek5Jn.png)

RECIPES INDEX JSON VIEW
![](https://i.imgur.com/UjMAhu0.png)

### 3. Must render at least one show page (show resource - 'one specific thing') via JavaScript and an Active Model Serialization JSON Backend.

The recipe show page is fetched using JSON and rendered through JavaScript on page load.  A user is also able to see the previous and next recipes dynamically without a page refresh.

RECIPES CONTROLLER
```
def show
  @recipe = Recipe.find(params[:id])
  respond_to do |format|
    format.html { render :show }
    format.json { render json: @recipe }
  end
end
```

RECIPE JS
```
function showRecipe() {
  if ($("#showRecipeData").length === 0) {
    return;
  }

  const recipeId = $("#showRecipeData").attr("data-id");

  $.get(`/recipes/${recipeId}.json`).done(loadRecipeData);
}

$(showRecipe);
```
```
function previousRecipe() {
  event.preventDefault();
  $(".js-prev").on("click", () => {
    let recipeId = $("#showRecipeData").attr("data-id");
    $.get(`/recipes/${recipeId}/previous.json`).done((json) => {
      loadRecipeData(json);
      $("#showRecipeData").attr("data-id", json["id"]);
    });
  });
}

$(previousRecipe);
```
```
function nextRecipe() {
  event.preventDefault();
  $(".js-next").on("click", () => {
    let recipeId = $("#showRecipeData").attr("data-id");
    $.get(`/recipes/${recipeId}/next.json`).done((json) => {
      loadRecipeData(json);
      $("#showRecipeData").attr("data-id", json["id"]);
    });
  });
}

$(nextRecipe);
```
```
function loadRecipeData(json) {
  const recipe = new Recipe(json);

  const recipeIngredients = recipe.ingredients.map((ingredient) => {
    return `
      <tr>
        <td>${ingredient.quantity}</td>
        <td>${ingredient.name}</td>
      </tr>
    `;
  });

  const comments = recipe.comments.map((comment) => {
    return `
      <li>
        ${comment.commenter} - ${comment.createdAt.toLocaleDateString()} - ${comment.body}
      </li>
    `;
  })

  const html = `
    <h1>${recipe.name}</h1>
    <h4>Created ${recipe.createdAt.toLocaleDateString()}<br> by
      <a href="/users/${recipe.user.id}">${recipe.user.name}</a></h4>
    <h3>About the ${recipe.name} cocktail</h3>
    <p>${recipe.description}</p>
    <h3>Ingredients in the ${recipe.name} cocktail</h3>
    <table>
      <thead>
        <tr>
          <th>Quantity</th>
          <th>Ingredients</th>
        </tr>
      </thead>

      <tbody>
        ${recipeIngredients.join('')}
      </tbody>
    </table>

    <h3>How to make the ${recipe.name} cocktail</h3>
    <p>${recipe.instructions}</p>

    <h3>Comments:</h3>

    <div>
      <p>${recipe.commentCountLabel()}</p>
    </div>

    <div><ul id="commentList">${comments.join('')}</ul></div><br>

    <form id="recipeCommentForm">
      <textarea name="comment[body]" id="comment_body"></textarea><br><br>
      <input type="submit" name="commit" value="Add Comment"><br><br>
    </form>
  `;

  $("#showRecipeData").html(html);
  setupCommentForm();
}
```

SHOW.HTML.ERB
```
<div>
  <%= button_tag "Previous Cocktail", class: "js-prev" %>
  <%= button_tag "Next Cocktail", class: "js-next" %>
</div>

<div id="showRecipeData" data-id="<%= @recipe.id %>"></div>

<%= render 'shared/links' %><br><br>
<%= link_to "Recipes by #{@recipe.user.name}", user_recipes_path(@recipe.user) %>
<%= link_to "Comments", recipe_comments_path(@recipe) %>

<% if current_user == @recipe.user %>
  <%= link_to "Edit Recipe", edit_recipe_path %><br><br>
  <%= form_for @recipe, url: recipe_path, method: :delete do |f| %>
    <%= f.submit "Delete" %>
  <% end %>
<% end %>
```

RECIPE SHOW VIEW
![](https://i.imgur.com/iWpch95.png)

### 4. Your Rails application must dynamically render on the page at least one serialized 'has_many' relationship through JSON using JavaScript.

As seen in the above images.  Users are able to comment on a recipe.  A recipe `has_many` comments and a comment `belongs_to` a recipe.

```
class RecipeSerializer < ActiveModel::Serializer
  attributes :id, :name, :description, :instructions, :created_at

  has_many :recipe_ingredients, key: :ingredients

  belongs_to :user
  has_many :comments

  def comments
    object.comments.by_date
  end
end
```
```
class CommentSerializer < ActiveModel::Serializer
  attributes :id, :body, :created_at

  belongs_to :recipe

  attribute(:commenter) {|o| o.object.user.name }
end
```

### 5. Must use your Rails application to render a form for creating a resource that is submitted dynamically and displayed through JavaScript and JSON without a page refresh.

In the previous Rails Project, a new comment for a recipe is created through nested routes such as recipes/4/comments/new.  For this project, the comment form now appears on the recipe show page.  A user can add a comment to a recipe, the comment is serialized and submitted via AJAX POST request.  The comment is displayed through JavaScript and JSON without a page refresh.  If the comment field is submitted without any input, an error message will be displayed.  Since all the information about a recipe's comments are now found on the recipe show page, all comment views and routes were deleted, except for the create method.

COMMENTS CONTROLLER
```
def create
  @recipe = Recipe.find(params[:recipe_id])
  @comment = @recipe.comments.build(comment_params)
  @comment.user = current_user
  if @comment.save
    render json: @comment, status: 201
  else
    render json: @comment.errors.full_messages, status: 422
  end
end
```

COMMENTS JS
```
function setupCommentForm() {
  const form = $("#recipeCommentForm");

  form.submit((event) => {
    event.preventDefault();

    const recipeId = $("#showRecipeData").attr("data-id");
    const values = form.serialize();
    const posting = $.post(`/recipes/${recipeId}/comments`, values);

    posting.done((data) => {
      showRecipe();
    }).error((response) => {
      alert(response.responseJSON.join('\n'));
    });
  });
}
```

Feel free to check out the application.

**Rails with JavaScript Application Link:**

[https://github.com/cookiemccormick/dark-and-stormy-js](http://)


