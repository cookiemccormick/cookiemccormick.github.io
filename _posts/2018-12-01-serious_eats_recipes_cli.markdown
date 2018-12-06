---
layout: post
title:      "Serious Eats Recipes CLI"
date:       2018-12-01 23:25:12 -0500
permalink:  serious_eats_recipes_cli
---


For my CLI Project I wanted to do something food related.  I LOVE FOOD!  My go-to site for any recipe is Serious Eats.  They have the most amazing tasting foods.  Seriously!

The homepage for [Serious Eats](https://www.seriouseats.com/) is a bit busy.  There are podcasts, videos, buying guides, techniques, ads, etc.  I found a page within the Serious Eats site that focuses on their recipe book - [The Food Lab](https://www.seriouseats.com/the-food-lab/recipes) - which is a much cleaner site to view recipes.

**I created three classes:**

1. class `Scraper` - responsible for scraping the website for all the recipes and for scraping the selected recipe website for their attributes.
2. class `Recipe` - main model representing an individual recipe.
3. class `CLI` - responsible for interacting with the user, starting and ending the application.

**The application:**

When the application starts, it fetches the index view and scrapes the list of recipes.  It then displays these recipes as a numbered list menu for the user to choose from.  Once the user selects a valid number from the list, that recipe's url is fetched and the complete details of the recipe are parsed into a recipe object.  Then it prints the details of the recipe - a
description, portion amount, active cooking time, total time, rating, ingredients, instructions and website.  The application will then ask the user if they would like to see another recipe.

If yes, the list of recipes will be displayed again and the user will be able to browse the selections.  If no, a friendly goodbye message is displayed.
