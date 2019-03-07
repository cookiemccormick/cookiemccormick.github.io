---
layout: post
title:      "Milk Memo - Sinatra Project"
date:       2019-03-07 01:48:28 +0000
permalink:  milk_memo_-_sinatra_project
---

For this project I had to think of something that is important to me, that I would enjoy making an app for.  I had a few ideas in mind, I love food, but my previous CLI project was about food so what else...  

Well, I'm currently 21 weeks pregnant with my first child.  I've been reading so much about babies and downloaded several pregnancy tracking apps, that I thought it would be fun to make my own.  

![](https://i.imgur.com/ugMJMdT.jpg)

## The Application
The application is called Milk Memo.  It tracks the user's pregnancy and their baby's development.  A user is able to signup for an account with a username, password, email and their baby's due date.  A user object and baby object are created upon signup.  Once logged in, the user can interact with their dashboard, where they can create, read, update and delete their upcoming appointments, notes and medications.  They may also edit their baby's name, gender and due date.  A countdown to the baby's birthdate is displayed on the dashboard as well as a picture of a fruit/vegetable that matches the approximate size of their baby that day.  The user is only able to edit their own data and cannot see any other user's data.

The appointment, baby, medicine and note classes are visible on the dashboard, where the user can READ all their information in one place.  There are separate views to CREATE and UPDATE/DELETE data for their appointments, baby, medicines and notes.  

The dashboard is customized to display the user's name and their baby's name.

Some CSS styling is present in the app and will continue to be a work in progess.  

Here is the dashboard view:

![](https://i.imgur.com/PSUvvUP.png)


## Challenges

USER SIGNUP - When a user signs up, their username, email, password and their baby's due date are taken.  A user is able to edit their baby's name, gender and due date after signup.  Users may want to signup for the app before knowing the baby's name and gender.  Also, I didn't want to overwhelm the user with too many data fields to fill out straight off the bat.  `NOT NULL` contraints were added to all required database columns to prevent `NULL` values.  For the baby's name, I took off the `NOT NULL` constraint and set the gender to `UNKNOWN` at the time of signup.  Gender constants were defined in the `Baby` model class and used in an ActiveRecord inclusion validation.

FLASH MESSAGE - I've incorporated flash messages that alert the user when an object is modified. They're also displayed when the user is not signed in or when required fields are left blank.  `rack-flash3` seems to be broken with this version of Sinatra.  After some digging, I swtiched to using `sinatra-flash`.

BABY COUNTDOWN - Upon signup the dashboard welcomes the user and presents a countdown of days remaining until the due date.  A user should be pregnant when signing up, but to prevent weird messages for someone who just wants to check out the app and signs up with bogus data, the app will check if their due date is within 40 weeks from the current day. If not, the app will just welcome the user.  If the 40 weeks is over, the app will congratulate the user on their new baby.

## Future Features
Milk Memo has room to grow.  Currently it tracks the progess of the pregnancy and baby's development prior to birth, but I think this app can further track the growth of the child through toddlerhood.  Possible ideas:

* Feedings
* Diaper changes
* Sleep schedule
* Milk
* Baby weight
* Social element where parents and chat with other parents
* Multiple babies

**Application Link:**

https://github.com/cookiemccormick/milk-memo


