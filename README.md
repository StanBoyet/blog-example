# Livecode session #1

We're gonna build a blog today.
The session is gonna be splited up in groups. Each group is the end of an accomplishment of some sort. It could be a new feature, a new page or a simple refactor. At the end of each group, we will commit our code and create a versioning.
It means that you can stop at the end of the bloc you want and still have something (somewhat) functionnal.

Let's go!

## Setup

Let's just a quick check, this command should give you your Rails version.
````
rails -v
````
If this doesn't work, please check your installation against the Rails Girls tutorial : http://guides.railsgirls.com/install

I'll be less verbose in this document from now on, the aim is just to keep track of the commands we ran and the objectives of each bloc, have fun.

## 1st bloc - App intialization

Call the Rails gem and ask to setup a new app in the folder `blog`

````
rails new blog
````

You should have a list of installed gems and at the near end something like :
````
Bundle complete! 15 Gemfile dependencies, 62 gems now installed.
````

Then let's go inside our new folder
````
cd blog/
````

And to finish this intialization, let's commit everything
````
git init
git add -A
git commit -am "rails app initialization"
````

You just got yourself a rails app! Let's do something with it now.

![you're on rails](http://i.imgur.com/jcIHcWD.png?1)


## 2nd bloc - The genesis of a blog post

We want a `post` to have
- a `title`, which is a `string` (because < 255 characters)
- a `content`, which is a `content` (because surely > 255 characters)

So let's use rails !

````
rails generate scaffold post title:string content:text
````

which responds with
````
      invoke  active_record
      create    db/migrate/20170115145705_create_posts.rb
      create    app/models/post.rb
      invoke    test_unit
      create      test/models/post_test.rb
      create      test/fixtures/posts.yml
      invoke  resource_route
       route    resources :posts
      invoke  scaffold_controller
      create    app/controllers/posts_controller.rb
      invoke    erb
      create      app/views/posts
      create      app/views/posts/index.html.erb
      create      app/views/posts/edit.html.erb
      create      app/views/posts/show.html.erb
      create      app/views/posts/new.html.erb
      create      app/views/posts/_form.html.erb
      invoke    test_unit
      create      test/controllers/posts_controller_test.rb
      invoke    helper
      create      app/helpers/posts_helper.rb
      invoke      test_unit
      invoke    jbuilder
      create      app/views/posts/index.json.jbuilder
      create      app/views/posts/show.json.jbuilder
      create      app/views/posts/_post.json.jbuilder
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/posts.coffee
      invoke    scss
      create      app/assets/stylesheets/posts.scss
      invoke  scss
      create    app/assets/stylesheets/scaffolds.scss
````

There we go. Now let's launch our app to see what we got, but first tell our database to integrate those changes!

````
bundle exec rake db:migrate
rails s
````

Then go to `http://localhost:3000/posts`, and we already have a ready to use app!
Close the server (CTRL + C) so we can continue the session, or open a new terminal & go to your app folder tab to continue.

Let's close this bloc now, and in the next one we will dig on what we just generated
````
git add -A
git commit -am "Scaffold of Post - Complete CRUD"
````

## 3rd bloc - What did we just do?

Let's start with our routes.
````
bundle exec rake routes
````

Which gives:
````
   Prefix Verb   URI Pattern               Controller#Action
    posts GET    /posts(.:format)          posts#index
          POST   /posts(.:format)          posts#create
 new_post GET    /posts/new(.:format)      posts#new
edit_post GET    /posts/:id/edit(.:format) posts#edit
     post GET    /posts/:id(.:format)      posts#show
          PATCH  /posts/:id(.:format)      posts#update
          PUT    /posts/:id(.:format)      posts#update
          DELETE /posts/:id(.:format)      posts#destroy
````

Let's open our project in our code editor. Here, I'm gonna use Atom
````
atom .
````
(*If this doesn't work, open Atom through your applications and Open the folder `blog`)*
And now let's open our routes configuration file `config/routes.rb`

While we're here, let's change our root route to our index of posts
````
# config/routes.rb
root to: "posts#index"
````

Let's follow the trail & go to `app/controllers/posts_controller.rb`
This is where the request go, and this is where the magic happens.

Our `root to: "posts#index"` sends the request straight to the `index` method.
Now that we saw how rails was getting all the informations, let's go to the view where we display it `app/views/posts/index.html.erb`
Change the title from `Posts` to `Our nice little blog`, and put it before the `notice` div, to have something like:
````erb
<h1>Our nice little blog</h1>
<p id="notice"><%= notice %></p>
````

We can witness that this is how the page is rendered. Go back to `http://localhost:3000/posts` and see that your title has changed \o/

Let's close this bloc which was more an explanation one.

````
git add -A
git commit -am "Update root route & posts index title"
````

But how do Rails now what a `post` is? Let's go to the model and have a look `app/models/post.rb`
This is empty! We will fix that in Bloc 5!

## 4th bloc - Let's create a user

Let's generate a user so our data model becomes `A user has_many posts`.

We don't need any controllers or views for our User (yet), so let's just generate a `model` and a database migration. We can do so with:
````
rails generate model User name:string
````

Which outputs these particular lines:
````
      create    db/migrate/20170115171817_create_users.rb
      create    app/models/user.rb
````

As you can see, we generated what we wanted.
One of those file is a database migration, so as usual, let's migrate!
````
bundle exec rake db:migrate
````

Open the rails console
````
rails console
````

Now that the console is opened, let's create our author :
````ruby
User.create(name: "Stan")
# Then close the console
exit
````

End of this bloc, a small commit won't do any harm ;)
````
git add -A
git commit -am "Create the User model"
````

## 5th bloc - All your posts are belong to me


User & Post don't know each other yet. So let's link them together, with our `A user has_many posts`.
First, open the `app/models/user.rb` file and write this line:
````ruby
has_many :posts
````

Then, open up `app/models/post.rb` file and write this:
````ruby
belongs_to :user
````

And lastly, let's generate a new column in our database to save the relationship:
````
rails generate migration AddUserToPosts user:references
````
*`references` here is telling to rails that user is not only a column but a reference to another table.*

The ouput is like this:
````
      invoke  active_record
      create    db/migrate/20170115173511_add_user_to_posts.rb
````

We created a new migration, but before migrating this one, let's check it out.
We see that Rails generated for us a migration with a `user_id` column to store the relation `belongs_to`.

````
bundle exec rake db:migrate
````

And let's fire up our console again and try it out!

````
rails console
````

And let's create a post with a user
````
alfred = User.create(name: 'Alfred')
post = Post.create(title: '2nd post!', content: 'Lorem ipsum is a great novel.', user: alfred )

#Let's make sure the two of them are linked
post.user
````

*Hint: have a little explanation about MVC*

This is what concludes our 5th bloc! Let's commit
````
git add -A
git commit -am "Link users to posts"
````
