# Helper Methods Part 2

## Getting started

In this project, you will continue to levelup your code, beginning with where we left off last time: `link_to`.

So navigate to `github.com/codespaces` (or reopen the previous lesson and use the "Load assignment" button) and reopen your `helper-methods-part-1-and-2` project codespace to continue building on what you accomplished in _Helper Methods Part 1_.

## Walkthrough video

<div class="bg-red-100 py-1 px-5" markdown="1">
**Please note**, the video is from a previous iteration of the project, so there are some differences:

- The project was previously called `helper-methods`, but it has been renamed to `helper-methods-part-1-and-2`
- I am using Gitpod as my cloud editor, so the interface looks a bit different.
- Anything contained in the project "README" is now contained in this Lesson
- I use a graphical user interface at the URL path `/git` to commit and push, _you_ should use [the VSCode built in workflow in this lesson](https://learn.firstdraft.com/lessons/50-git-commit-and-push)
- I use `bin/server` to start my live app preview, _you_ should use `bin/dev`
</div>

Did you read the differences above? Good! Then [here is a walkthrough video for this project.](https://share.descript.com/view/TrSygtEDGsx)

**As you watch the video, pause it frequently, read the associated text, and type out the code.**

## `form_with`

It's time to work on our `<form>`s and turn those into helper methods also. This is going to pay big dividends for us down the line.

Let's start with the form in `app/views/movies/new.html.erb`. The new helper method we'll use to replace this whole form is `form_with`. Let's begin:

```erb{9-11}
<!-- app/views/movies/new.html.erb -->

<h1>New movie</h1>

<% @the_movie.errors.full_messages.each do |message| %>
  <p style="color: red;"><%= message %></p>
<% end %>

<%= form_with(url: movie_path(@the_movie)) do %>
  
<% end %>

<form action="<%= movie_path(@the_movie))%>" method="post">
  <input name="authenticity_token" value="<%= form_authenticity_token %>" type="hidden">
  <!-- ... -->
```

As you can see, the method takes arguments. In this case we pass the option `url:` the output of our route helper `movie_path(@the_movie)` (which we know is just `"/movies"`). We also put this helper in a `do`-`end` block, because it's going to write a form for us in this block!

In the live app, refresh the `/movies/new` page and "view source". You should see that `form_with` has produced:

```erb
<form action="/movies" accept-charset="UTF-8" method="post"><input type="hidden" name="authenticity_token" value="some-long-token">

</form>
```

Compare that with the form that we wrote by hand. Pretty similar right? 

The `form_with` helper not only added an attribute with the accepted character set (UTF-8), but also automatically created an authenticity token to protect our form against CSRF attacks. We don't need to worry about remembering this step with the `form_with` helper. 

Let's move all of our previous inputs and tags into this block now (removing our old `<form>` opening and closing tags):

```erb{5-23}
<!-- app/views/movies/new.html.erb -->

<!-- ... -->
<%= form_with(url: movie_path(@the_movie)) do %>
  <div>
    <label for="title_box">
      Title
    </label>

    <input type="text" id="title_box" name="query_title" value="<%= @the_movie.title %>">
  </div>

  <div>
    <label for="description_box">
      Description
    </label>

    <textarea id="description_box" name="query_description" rows="3"><%= @the_movie.description %></textarea>
  </div>

  <button>
    Create Movie
  </button>
<% end %>
<!-- ... -->
```

Now what else can we do here?

For `<labels>`, we can use `label_tag`:

```erb{3}
<%= form_with(url: movie_path(@the_movie)) do %>
  <div>
    <%= label_tag :title_box, "Title" %>

    <input type="text" id="title_box" name="query_title" value="<%= @the_movie.title %>">
  </div>
```

The first argument to `label_tag` is the `for=""` attribute that connects the label with the input (by the input `id=""`), and the second argument is the copy we want shown. As usual, refresh and view source on the live app, and you should see a `<label>` tag rendered by this helper method. Same as before!

We can actually leave out the first argument `:title_box`, and just write `<%= label_tag "Title" %>`. What does that do on the page? View source and see for yourself.

The `for=""` attribute is now populated with the copy (`for="Title"`). That's nice, but in this case, we want to make sure our `for` and `id` attribute values matchup, so let's leave it as `<%= label_tag :title_box, "Title" %>`.

We also have helper methods for our `<input>`s. You might expect it to be `input_tag`, if we're following the same kind of pattern, but it's actually not because we have a separate helper method for each type of input: 

```erb{5}
<%= form_with(url: movies_path) do %>
  <div>
    <%= label_tag :title_box, "Title" %>

    <%= text_field_tag :query_title, @the_movie.title %>
  </div>
```

The `text_field_tag` that's replacing our `<input type="text"...>` HTML element is taking the `name=""` attribute as the first argument. This `name=""` is how the input will be registered in the `params` hash (in this case, as `query_string`), so that's a very important argument. After that, we have the `value=""` attribute that we want to prepopulate our form with.

Refresh the live app and view source again. What do you notice about the `id=""` attribute with this input helper method?

It is just using the `name=""` attribute `query_string`, that we supplied the method! Similar to leaving out the `for=""` attribute on the `label_tag` helper. Rails will try to fill these in for us. We'll see how useful this is soon.

But for now, let's note that the final argument to our `text_field_tag` can be a hash of additional options that I want:

```erb{5:(54-75)}
<%= form_with(url: movies_path) do %>
  <div>
    <%= label_tag :title_box, "Title" %>

    <%= text_field_tag :query_title, @the_movie.title, { id: "title_box" } %>
  </div>
```

And refreshing our `/movies/new` then viewing source, we can see that added an `id="title_box"` to our `<input>` HTML element.

Do the same for the movie description as well. This is a `<textarea>` input, so we need to use the helper `text_area_tag`. Also, we had a `rows="3"` attribute, so we need to add that to our optional hash. Otherwise, the rest is similar to the "Title", so the whole form should end up like:

```erb{12,14}
<!-- app/views/movies/new.html.erb -->

<!-- ... -->
<%= form_with(url: movie_path(@the_movie)) do %>
  <div>
    <%= label_tag :title_box, "Title" %>

    <%= text_field_tag :query_title, @the_movie.title, { id: "title_box" } %>
  </div>

  <div>
    <%= label_tag :description_box, "Description" %>

    <%= text_area_tag :query_description, @the_movie.description, { id: "description_box", rows: 3 } %>
  </div>

  <button>
    Create Movie
  </button>
<% end %>
<!-- ... -->
```

Check out the live app. Everything looking good? Great!

The last thing is the `<button>`. Even for this, we have a `button_tag` helper method:

```erb
  <!-- <button>
    Create Movie
  </button> -->
  <%= button_tag "Create Movie" %>
```

All right, so now we've replaced all of the essential parts of this form, other than `<div>`s that we use to organize it, with helper methods. Ultimately, the output is the _exact_ same HTML as before, but it's better in many ways.

The most concrete way in this example is that we didn't have to generate the CSRF token manually. That's going to save us a lot of work over the hundreds of forms that we're going to make! 

Make a git commit now.

## Update Edit Form 00:12:00 to 00:16:00

Let's take our new helper methods and do the same update on our edit page. Try to type out the new form to build some muscle memory. It's almost identical to our `new.html.erb` form. However, we can see a few key differences that need to be incorporated:

```erb
<!-- app/views/movies/new.html.erb -->

<h1>Edit movie</h1>

<% @the_movie.errors.full_messages.each do |message| %>
  <p style="color: red;"><%= message %></p>
<% end %>

<%= form_with(url: movie_path(@the_movie), method: :patch) do %>
  <div>
    <%= label_tag :title_box, "Title" %>

    <%= text_field_tag :query_title, @the_movie.title, {id: "title_box" } %>
  </div>

  <div>
    <%= label_tag :description_box, "Description" %>

    <%= text_area_tag :query_description, @the_movie.description, {id: "description_box", rows: 3 } %>
  </div>

  <%= button_tag "Update Movie" %>
<% end %>
```
{: mark_lines="9"}

The default method that HTML forms use is `get`. But Rails is smart, and it knows that many of our forms use `post`. So by default, the `form_with` method on our `new.html.erb` form added the `method="post"` attribute _and_ the authenticity token. 

Similarly, we can tell `form_with` exactly which method (HTTP verb) we want to use. In the case of our `edit.html.erb` form, we actually want the `method: :patch`, which we pass in as an argument above.

If you view source on the new edit form, you will see that adding this argument to the helper method produced that odd bit of HTML to "trick" the form into making a patch request:

```erb
<input name="_method" value="patch" type="hidden">
```

We never need to remember to add that again, because `form_with(..., method: :patch)` does it for us. Nifty!

Does the edit form work like before we refactored? If yes, then make another git commit!

## Refactoring `MoviesController` 00:16:00 to 00:24:30

Our views are now much more concise. We're using Ruby helper methods embedded in the view templates to generate HTML in a more secure way, and in a more reusable way using route helper methods.

Now it's time to spend some time refactoring our `app/controllers/movies_controller.rb` file. If you haven't already, go through and make all the syntax changes. For instance, a line like:

```ruby
matching_movies = Movie.where({ :id => the_id })
```

should end up like:

```ruby
matching_movies = Movie.where(id: the_id)
```

In AD1, we were very deliberate in the three steps to get an `ActiveRecord` instance (a single row of a table) compared to an `ActiveRecord:Relation` (many rows of a table):

```ruby
  ...
  def show
    the_id = params.fetch(:id) # FIRST we get the ID

    matching_movies = Movie.where(id: the_id) # THEN we get the set of matching rows (ActiveRecord:Relation)

    @the_movie = matching_movies.first # FINALLY we get one instance of ActiveRecord, or one row
  end
  ...
```

As you might expect, professional Rails, developers do not do this. Instead, you will often see:

```ruby
  ...
  def show
    @the_movie = Movie.where(id: params.fetch(:id)).first
  end
  ...
```

There's actually a succinct method that is designed to do exactly that. It's called `find_by`, and it's exactly the same thing as `.where`, except that if automatically returns just the `.first` record given a collection:

```ruby
  ...
  def show
    @the_movie = Movie.find_by(id: params.fetch(:id))
  end
  ...
```

That method will return a `nil` if the `id` isn't found. (Try it in the `rails console` to see for yourself.)

And we can actually go one step farther and use another method:

```ruby
  ...
  def show
    @the_movie = Movie.find(params.fetch(:id))
  end
  ...
```

The `find` method, only takes an integer argument, and it assumes that you are searching on the table's ID column. No need to tell it which column as we did with `.where`. 

As opposed to `find_by`, which returns `nil` if the record with the given ID doesn't exist, `find` throws an error exception of class `ActiveRecord::RecordNotFound`. (Again, use the `rails console` to experiment with both methods on the `Movie` model.)

Once I push this my app to Heroku, if I'm using the `find` method, then this error message shows up as a 404 page, which is the correct behavior when someone tries to visit a resource that doesn't exist (like `/movies/890909820`, or some ID number we don't have in our table). If we used `find_by` and the query returned a `nil`, then a 500 page would be shown ("Something went wrong"), which is not the experience we want for our users.

And another thing while we're talking about this `show` method. Conventionally, Rails developers don't say `the` underscore `movie`. We did that in AD1 to be very explicit. The convention is to name these variables the same thing as the class name and the controller name!

```ruby
  ...
  def show
    @movie = Movie.find(params.fetch(:id))
  end
  ...
```

You _can_ name your variables whatever you want to, but it's just a convention and you'll see this done a lot.

Similarly, we should rename all of our `list_of_movies` variables to just `movies` (the resource plural). 

For instance, the `index` action should end up like:

```ruby
  ...
  def index
    @movies = Movie.order(created_at: :desc)

    respond_to do |format|
      format.json do
        render json: @movies
      end

      format.html
    end
  end
  ...
```

## Query String Naming and View Refactoring 00:24:30 to 00:30:00

Now what about those query strings? 

```ruby
  ...
  def create
    @movie = Movie.new
    @movie.title = params.fetch("query_title")
    @movie.description = params.fetch("query_description")

    if @movie.valid?
      @movie.save
      redirect_to movies_path, notice: "Movie created successfully."
    else
      render template: "movies/new.html.erb"
    end
  end
  ...
```
{: mark_lines="4-5"}

Recall, `"query_title"` is the `name` we gave to the input in our form, which goes into query string, and then into the `params` hash that we're fetching from. But we don't need the `"query_"` construction, it was just there to remind us. 

If we want to be more professional, we would just change those to the name of the column we are getting inputs for:

```ruby
  ...
  def create
    @movie = Movie.new
    @movie.title = params.fetch("title")
    @movie.description = params.fetch("description")

    if @movie.valid?
      @movie.save
      redirect_to movies_path, notice: "Movie created successfully."
    else
      render template: "new"
    end
  end
  ...
```
{: mark_lines="4-5"}

And, also, in the `params` hash, we typically fetch on symbols, rather than strings. We can only use strings and symbols interchangably in `params` because it is a special subclass of `Hash`:

```ruby
  ...
  def create
    @movie = Movie.new
    @movie.title = params.fetch(:title)
    @movie.description = params.fetch(:description)

    if @movie.valid?
      @movie.save
      redirect_to movies_path, notice: "Movie created successfully."
    else
      render template: "new"
    end
  end
  ...
```
{: mark_lines="4-5"}

Now, what that `params` key has to match is the `name` that we assigned that input back on our form. Let's take a peek back there on the `new.html.erb` form:

```erb
<!-- app/views/movies/new.html.erb -->

...
<%= form_with(url: movie_path(@movie)) do %>
  <div>
    <%= label_tag :title_box, "Title" %>

    <%= text_field_tag :query_title, @movie.title, {id: "title_box" } %>
  </div>

  <div>
    <%= label_tag :description_box, "Description" %>

    <%= text_area_tag :query_description, @movie.description, {id: "description_box", rows: 3 } %>
  </div>
...
```
{: mark_lines="4 8 14"}

First of all, note that we changed `@the_movie` to `@movie`, because we changed the name of that instance variable in the controller. Now, look at these changes to align our view template with our controller `params` fetching:

```erb
<!-- app/views/movies/new.html.erb -->

...
<%= form_with(url: movie_path(@movie)) do %>
  <div>
    <%= label_tag :title, "Title" %>

    <%= text_field_tag :title, @movie.title, {id: "title" } %>
  </div>

  <div>
    <%= label_tag :description, "Description" %>

    <%= text_area_tag :description, @movie.description, {id: "description", rows: 3 } %>
  </div>
...
```
{: mark_lines="6 8 12 14"}

We don't need the `query_` and we don't need the `_box`. Those are names we made up to help use keep track of things, but they aren't used in a professional code base.

Also, we can remove some more things here and let Rails automatically: 
 
 - set the label copy (by calling `.capitalize` on the `:title` or `:description`) 
 - and set the `for=""` / `id=""` attributes (which are just going to default to `"title"` and `"description"`):

```erb
<!-- app/views/movies/new.html.erb -->

...
<%= form_with(url: movie_path(@movie)) do %>
  <div>
    <%= label_tag :title %>

    <%= text_field_tag :title, @movie.title %>
  </div>

  <div>
    <%= label_tag :description %>

    <%= text_area_tag :description, @movie.description, { rows: 3 } %>
  </div>
...
```
{: mark_lines="6 8 12 14"}

Spend a few minutes now manually testing your app and chasing down any error messages caused by the controller refactoring. This comes down to changing the `list_of_movies` to `movies` (plural) and any variable that includes `the_movie` or `a_movie` (see the `index.html.erb` loop!) to just be `movie`. 

In AD1, we named every variable differently to be very explicit about the connections between our pages and actions. But, now that we understand the conventions, which is to just name the variable after the resource (`movie` or `movies`), we don't have to think about making up different names.

That was a lot of refactoring. Let's make a git commit. 

## Bundled Subhashes in `params` 00:30:00 to 00:39:00

Let's do some more work on our forms. We can demonstrate something with some dummy code, maybe in the `index` page:

```erb
<!-- app/views/movies/index.html.erb -->

<h1>
  List of all movies
</h1>

<form>
  <input name="zebra">
  <button>Submit</button>
</form>
...
```
{: mark_lines="7-10"}

Now, if we refresh `/movies`, we should see our blank form with just one, un-labelled input box. If we type "hi" into the form and click "Submit", then go to our server log in the terminal (the terminal that we ran `bin/server` in that's connected to the live app), we should be able to find the `params` hash and see:

```bash
Parameters: {"zebra"=>"hi"}
```

What happens if I change my form slightly to add some square brackets next to `"zebra"`:

```erb
<!-- app/views/movies/index.html.erb -->

<h1>
  List of all movies
</h1>

<form>
  <input name="zebra[]">
  <button>Submit</button>
</form>
...
```
{: mark_lines="8"}

Again, refresh `/movies` and submit something to the blank form again. How do the `params` look? You should see something like:

```bash
Parameters: {"zebra"=>["hi"]}
```

The key is still `"zebra"`, but now it put the value into an array. Interesting. Let me put in another one of these inputs:

```erb
<!-- app/views/movies/index.html.erb -->

<h1>
  List of all movies
</h1>

<form>
  <input name="zebra[]">
  <input name="zebra[]">
  <button>Submit</button>
</form>
...
```
{: mark_lines="9"}

And on the form put something different in both input boxes then submit, and view the `params` in the server log. You should see something like:

```bash
Parameters: {"zebra"=>["hi", "there"]}
```

Rails put the two values for the two different inputs on the same `name`d input fields into an array. So now, we could do `params.fetch("zebra")` and that would return an array that we could do `.each` on to loop through. In other words, we have created nested structures in our `params`.

This gives you a lot of different power and flexibility. A classic use case here is creating checkboxes. For instance with code like this:

```erb
...
<form>
  <label>Red</label>
  <input type="checkbox" name="zebra[]" value="red">
  
  <label>Blue</label>
  <input type="checkbox" name="zebra[]" value="blue">

  <button>Submit</button>
</form>
...
```

Which would lead to the `params` hash: `{"zebra"=>["red", "blue"]}`.

What else? Remove the `type="checkbox"` to go back to the default `"text"` input, and add something else to the brackets:

```erb
<!-- app/views/movies/index.html.erb -->

<h1>
  List of all movies
</h1>

<form>
  <input name="zebra[giraffe]">
  <input name="zebra[elephant]">
  <button>Submit</button>
</form>
...
```
{: mark_lines="8-9"}

Type something into both inputs on the `/movies` form in your live app. What do you think the. What do you think the `params` hash is going to look like? Take a guess. Is it going to be an array again with values `giraffe` and `elephant` in it?

Have a look at the server log (assuming you typed "hi" and "there" into the boxes on the form):

```bash
Parameters: {"zebra"=>{"giraffe"=>"hi", "elephant"=>"there"}}
```

Now the value to the `"zebra"` key is also a hash; a nested hash! And the first key for the first input is `"giraffe"`. It has a value of whatever I typed. The second key is `"elephant"`, and it also has a value of what I typed. 

This is a very handy way of bundling together related inputs in a form into a labeled **subhash**. 

This is the technique that we use more often when we're building forms. All of the inputs that are related to one model object, a `Movie` or a `Director`, are bundled into one subhash and stored a key `movie` or `director`. We want _that_ resoure to be a top level key in the `params` hash and then the subhash should have all of the attributes (or column values) for that top level key. 

## Refactor Forms with Mass Assignment 00:39:00 to 00:49:00

Given this new principle of nested hashes in our `params`, let's reorganize our forms. Open the `new.html.erb` form:

```erb
<!-- app/views/movies/new.html.erb -->

...
<%= form_with(url: movie_path(@movie)) do %>
  <div>
    <%= label_tag :title %>

    <%= text_field_tag :title, @movie.title %>
  </div>

  <div>
    <%= label_tag :description %>

    <%= text_area_tag :description, @movie.description, { rows: 3 } %>
  </div>
...
```

My goal with this form, is to have the `params` hash end up looking like:

```ruby
{ :movie => { :title => "Some title", :description => "Some description" } }
```

Whereas, right now, it looks like:

```ruby
{ :title => "Some title", :description => "Some description" }
```

So let's change the `"name"` attribute in the helper methods to use those square brackets we just learned about:

```erb
...
<%= form_with(url: movie_path(@movie)) do %>
  <div>
    <%= label_tag :title %>

    <%= text_field_tag "movie[title]", @movie.title %>
  </div>

  <div>
    <%= label_tag :description %>

    <%= text_area_tag "movie[description]", @movie.description, { rows: 3 } %>
  </div>
...
```
{: mark_lines="6 12"}

Now we can test our form at **/movies/new** and we will get an error because we haven't changed the controller yet to fetch the parameters in this new format. But, if we look at the server log, then we should see that we achieved our goal and we have subhash. 

Let's update our controller to take advantage of this in the `create` action: 

```ruby
# app/controllers/movies_controller.rb

  ...
  def create
    @movie = Movie.new
    @movie.title = params.fetch(:movie).fetch(:title)
    @movie.description = params.fetch(:movie).fetch(:description)
  ...
```
{: mark_lines="6-7"}

Now the form on **/movies/new** should work.

But with our new subhash, we can be slightly more concise. Suppose you have a `Movie` object, and you happen to have a hash laying around that has keys in it that exactly match your column names. So the hash has _exactly_ the same column names as the `movies` table.

Well, I can say `Movie.new`, and I can give the `.new` method that hash as an argument: 

```ruby
# app/controllers/movies_controller.rb

  ...
  def create
    movie_attributes = params.fetch(:movie)
    @movie = Movie.new(movie_attributes)
    # @movie.title = params.fetch(:movie).fetch(:title)
    # @movie.description = params.fetch(:movie).fetch(:description)
  ...
```
{: mark_lines="5-6"}

The `.new` method will iterate through the key value pairs in `movie_attributes`: `{ :title => "Some title, :description => "Some description }`, and it will assign this value to the matching column! 

We don't need to have the assignment happening on separate lines. Imagine if our table had 30 columns we were trying to assign? This **mass assignment** technique would help a lot.

Is this going to work? Try and refresh **/movies/new** and try the form again... Unfortunately, it's not going to work. It's not that easy, my friends. 

I get a `ForbiddenAttributesError`. What is this? This is Rails doing more fancy security on our behalf without us even realizing it.

There's an attack where people manipulate forms, which already have that CSRF authenticity token in them. A malicious user can still manipulate what inputs are in the form to put inputs that are not supposed to be there (e.g., `admin`), allowing them to modify columns that are not supposed to be modified.

How do we fix this error? We have to say which attributes are _allowed_ to be mass assigned by **whitelisting** them.

<aside markdown="1">
The security researcher who discovered this security hole got attention for it by hacking GitHub, making himself an admin on the Rails repository, and then making a commit to Rails itself. That got their attention! And the whitelisting fix was added.
</aside>

In our `create` action, we need to change the way we get the `params`:

```ruby
# app/controllers/movies_controller.rb

  ...
  def create
    movie_attributes = params.require(:movie).permit(:title, :description)
    @movie = Movie.new(movie_attributes)
    # @movie.title = params.fetch(:movie).fetch(:title)
    # @movie.description = params.fetch(:movie).fetch(:description)
  ...
```
{: mark_lines="5"}

Rather than `.fetch`ing `:movie` from the `params`, we need to `require(:movie)` and then `.permit` and list of the allowed columns. 

Whenever you add a new column, you also have whitelist it in your **strong parameters** list and then it'll work. Try the **/movies/new** form and see.

What if you added a new column or forgot about one of them, like if you remove `:description` from the `permit` list above? Try it and check the server log. You should see in red font `Unpermitted parameter: :description`. Rails makes this error really prominent, because it's so common. Don't forget to whitelist all of the columns you want to modify, but, if you do, you'll be reminded!

Please take a moment and refactor the `update` action as well to follow this new format. When you're done, make a **/git** commit.

## Form Builder with Model 00:49:00 to 01:06:00

We need to go back to our forms and make sure the `for=""` and `id=""` attributes matchup between labels and inputs. They already did before, but when we introduced that `[]` square bracket syntax to get to mass assignment with whitelisting, we went a bit backwards:

```erb
<!-- app/views/movies/new.html.erb -->

<h1>New movie</h1>

<% @movie.errors.full_messages.each do |message| %>
  <p style="color: red;"><%= message %></p>
<% end %>

<%= form_with(url: movies_path) do %>
  <div>
    <%= label_tag :title %>

    <%= text_field_tag "movie[title]", @movie.title %>
  </div>

  <div>
    <%= label_tag :description %>

    <%= text_area_tag "movie[description]", @movie.description, { rows: 3 } %>
  </div>

  <%= button_tag "Create Movie" %>
<% end %>
```
{: mark_lines=""}

Well, there's actually a better way if we want that nested hash behavior. If we're making a form for the specific purpose of creating a record in our database table or updating a record with `ActiveRecord`, there's a more specific way of using `form_with`. Here's how it works. 

Instead of saying `form_with(url: ...)`, you say `form_with(model: ...)`. And the argument for this should be the object _itself_, that you're trying to build a form for! 

```erb
<!-- app/views/movies/new.html.erb -->

<h1>New movie</h1>

<% @movie.errors.full_messages.each do |message| %>
  <p style="color: red;"><%= message %></p>
<% end %>

<%= form_with(model: @movie) do %>
  <div>
    <%= label_tag :title %>

    <%= text_field_tag "movie[title]", @movie.title %>
  </div>

  <div>
    <%= label_tag :description %>

    <%= text_area_tag "movie[description]", @movie.description, { rows: 3 } %>
  </div>

  <%= button_tag "Create Movie" %>
<% end %>
```
{: mark_lines="9"}

Now, what the heck is this `@movie` object? 

Remember how we created a new `@movie` object in the `new` action in our controller? We did that only because we were going to draw the errors collection in the first few lines on the `new.html.erb` page. We created a blank object just to satisfy the case where a user visits **/movies/new** for the first time, and there is no prepopulating of fields to do. 

Now `@movie` is coming in handy again, by telling the form that we want to `create` this new empty movie object. We want to take this empty thing and save it to our database.

Okay, with only that change, what's going to happen to the `action` attribute on the form?

Refresh **/movies/new** and view source. It did the same exact thing! The `action` is still `"/movies"`. 

This is another one of these so-called "magical" instances. Rails knows, or assumes, that there is a route called `@movie.class.downcase` (`movie`) that is an alias for`"/movies"`, and it can then call `movie_url` and find the action associated with that route in `config/routes.rb` (`create`) to build a form for the new movie!

Madness, right? This is the result of all of these compounding dividends, straight from our RESTful routes to here!

Another benefit of using `form_with(model: ...)` is that there's now a block variable, known as a **form builder object**. 

```erb
<!-- app/views/movies/new.html.erb -->

<h1>New movie</h1>

<% @movie.errors.full_messages.each do |message| %>
  <p style="color: red;"><%= message %></p>
<% end %>

<%= form_with(model: @movie) do |form| %>
  <div>
    <%= label_tag :title %>

    <%= text_field_tag "movie[title]", @movie.title %>
  </div>

  <div>
    <%= label_tag :description %>

    <%= text_area_tag "movie[description]", @movie.description, { rows: 3 } %>
  </div>

  <%= button_tag "Create Movie" %>
<% end %>
```
{: mark_lines="9"}

Conventially, we just call it `form`. Now that `form` object has methods called `.label`, `.text_field`, etc., which we can use like so:

```erb
<!-- app/views/movies/new.html.erb -->

<h1>New movie</h1>

<% @movie.errors.full_messages.each do |message| %>
  <p style="color: red;"><%= message %></p>
<% end %>

<%= form_with(model: @movie) do |form| %>
  <div>
    <%= form.label :title %>

    <%= form.text_field :title %>
  </div>

  <div>
    <%= form.label :description %>

    <%= form.text_area :description, { rows: 3 } %>
  </div>

  <%= form.button %>
<% end %>
```
{: mark_lines="11 13 17 19 22"}

We dropped all our syntax for the nested mass assignment subhash, because form is associated with the `@movie` object, which is an instance of `Movie`! We also dropped references to `@movie.title` and `@movie.description`, again because we already have the reference to `@movie` in the form builder. 

Finally, we didn't even need to put any "Create Movie" copy on the `form.button` method, because Rails will check and see that the object does not exist in the database yet (it has not been **persisted**), so it will choose the copy "Create Movie" for us.

Wow, Rails magic indeed.

Check the **/movies/new** form functionality and also view source on the page. You should still see all the syntax and attributes we spent time writing on our own, now done in a few succinct lines of code by Rails!

This is the point. Rails will save you a ton of time with default functionallity if you name things the way we expect. You can override any of these features, but aren't they nice?

## Challenge 01:03:00 to 01:06:00

My challenge to you: refactor the `edit` form and the `update` action to match what we've done here with the `new` form and `create` action. After that, generate a new model, say `directors`:

```bash
rails g model director name ...
```

Give it a few more columns (maybe date of birth, bio, etc.). 

Starting from only this model (not a scaffold) try to build up the entire resource. All of the routes. The seven golden actions in the controller that we did here. But do it all in this modern way.

And one last thing. Because these seven "golden" routes are so common, there's actualy a shortcut to create them all:

```ruby
# config/routes.rb

Rails.application.routes.draw do
  root "movies#index"

  resources "movies"

  # # CREATE
  # post "/movies" => "movies#create", as: :movies # movies_url and movies_path 
  # get "/movies/new" => "movies#new", as: :new_movie # new_movie_url and new_movie_path
          
  # # READ
  # get "/movies" => "movies#index" # we defined a method for this path 4 lines above
  # get "/movies/:id" => "movies#show", as: :movie # movie_url and movie_path
  
  # # UPDATE
  # patch "/movies/:id" => "movies#update" # we defined a method for this path 4 lines above
  # get "/movies/:id/edit" => "movies#edit", as: :edit_movie # edit_movie_url and edit_movie_path
  
  # # DELETE
  # delete "/movies/:id" => "movies#destroy" # we defined a method for this path 4 lines above

  #------------------------------
end
```
{: mark_lines="6"}

That's it. We can comment out all seven routes, including the named route helper methods we defined after `as:`, and just write `resources "movies"` and all those routes and helper methods will be defined in our app! Boom.
