---
layout: post
title: "Expressing intent via pattern matching"
tags: [elixir, intent, pattern, matching]
---

Pattern matching. It's Elixir's one of the most simple yet powerful features. 
Most of us, at least at the beginning of the Elixir journey, think of it as 
assignment. And we discover it is not assigment and yet we still use it as one. 
But, pattern matching is (and can be) much more than binding a value to a variable. 

**Disclaimer:** While this is cool to me and I think I've done a good job, keep in 
mind that I am a Elixir newbie. So, this might be stupid to some of you and/or you
may know a better way to do it. In whichever case, feel free to drop a comment in 
an **educational sense**.

## Express intent

As an engineer at [Evercam](http://evercam.io), I am migrating an older Rails API 
over to Phoenix. Since I am very fresh on the team and the domain is new to me, 
I started with a simple endpoint - `POST /v1/users`, or in other words, 
creating a user.

Now, the logic in the endpoint controller is quite simple. It creates a `User` 
changeset with the params and checks if it is valid. Then, it saves the user. 
If all of that is completed, the record is returned as a JSON. Also, based on 
the older API, there are certain HTTP codes that the API must return (as you can 
see below). 

Take note that this is not perfect, but it's good enough for an example.

After the first try, this is what I came up with:

{% highlight elixir %}
defmodule Evercam.UserController do
  use Evercam.Web, :controller

  def create(conn, %{ "user" => user_params }) do
    user_changeset = User.changeset(%User{}, user_params)

    if user_changeset.valid? do
      case Repo.insert(user_changeset) do
        { :ok, user } ->
          conn
          |> put_status(:created)
          |> render("user.json", %{ user: user })
        { :error, changeset } ->
          conn |> handle_error(:conflict, changeset)
      end
    else
      conn |> handle_error(:bad_request, user_changeset)
    end
  end

  defp handle_error(conn, status, changeset) do
    conn
    |> put_status(status)
    |> render(Evercam.ChangesetView, "error.json", changeset: changeset)
  end
end
{% endhighlight %}

Now, the tests were passing and all, but this felt dirty to me. I mean, the code
worked but something just didn't feel right. So, since I got the tests green, I 
wanted to try to refactor this. 

## Motivation to refactor

The first thing that I aimed for was extracting the logic of building the user out 
of the controller itself. As I've learnt from my colleagues and from
the [Programming Phoenix](https://pragprog.com/book/phoenix/programming-phoenix) 
book, it's a good practice to keep _impure_ code separate. 
If you are wondering, impure code means code that has side effects. Like, for example,
writing to the database. 

I don't want to use the "fat models - skinny controller" analogy here, but, it's
kind of similar. I didn't want to introduce more "weight" to the `User` model, 
but I was interested of moving things out to a service module of some sort. 

Keeping the controller _pure_ is a good practice and will reduce the cognitive load
of reading the controller code in the future.

## Refactoring

So, this is what I came up with:

{% highlight elixir %}
defmodule Evercam.UserSignup do
  alias Evercam.Repo

  def create(user_changeset) do
    if user_changeset.valid? do
      case Repo.insert(user_changeset) do
        { :ok, user } -> { :success, user }
        { :error, changeset } -> { :duplicate_user, changeset }
      end
    else
      { :invalid_user, user_changeset }
    end
  end
end
{% endhighlight %}

Now, as you can see, the code is quite similar. But, the beauty of this, in my 
opinion, are the tuples that the `create/1` function it returns. 

Why? Well, let's see the controller code now:

{% highlight elixir %}
defmodule Evercam.UserController do
  use Evercam.Web, :controller

  def create(conn, %{ "user" => user_params }) do
    user_changeset = User.changeset(%User{}, user_params)

    case Evercam.UserSignup.create(user_changeset) do
      { :invalid_user, changeset } ->
        handle_error(conn, :bad_request, changeset)
      { :duplicate_user,  changeset } ->
        handle_error(conn, :conflict, changeset)
      { :success, user } ->
        conn
        |> put_status(:created)
        |> put_resp_header("access-control-allow-origin", "*")
        |> render("user.json", %{ user: user })
    end
  end

  defp handle_error(conn, status, changeset) do
    conn
    |> put_status(status)
    |> put_resp_header("access-control-allow-origin", "*")
    |> render(Evercam.ChangesetView, "error.json", changeset: changeset)
  end
end
{% endhighlight %}

As you can see, since the controller returns different HTTP status codes for 
every type of error, with the refactor is really easy to see why a certain code will
be returned. 

For example, if you are wondering why a test that you just wrote fails with a HTTP 400, 
it will be clear to you that the signup process failed because the `UserSignup.create/2`
function returned `:invalid_user`. This can mean only one thing - there is something
wrong with the data in the `user_changeset`.

As you can see, this definitely makes things a bit more compact and reasoning about
the endpoint flow is easier. 
