In Rails and other MVC frameworks, you don't always need to fill out an entire form to update a resource. Similar to the 'destroy' function in many a RESTful application, it's pretty easy to dedicate a route to alter a single attribute in a given model. By doing so, a user can update that information with a single button (or link, using jQuery) making a `PATCH` request.
<!--more-->

My most recent project features a fairly lightweight admin scope where I grant a user managerial prvileges using a simple boolean value, `admin`. The boolean is set to `false` by default in the migration and model. The user index page, which is restricted to admin users, allows an admin user to grant or revoke admin access to any user aside from themselves with a single click: 

![Users view](/assets/images/all_users.png)

How was this done? First, in `/app/controllers/users_controller.rb`, an action called `admin_toggle` was defined as such:

```ruby
...
before_action :authenticate_user!
before_action :admin_only, only: %i[index update admin_toggle destroy]
...

def admin_toggle
  set_user
  if @user == current_user
    flash[:notice] = "Can't revoke own admin status"
  elsif @user&.admin?
    @user.admin = false
    flash[:notice] = "#{@user.full_name} is no longer an admin"
  elsif @user&.admin == false
    @user.admin = true
    flash[:notice] = "#{@user.full_name} is now an admin"
  end
  @user.save
  redirect_to users_path
end
```

This action is only accessible to admin users through a helper, and will toggle the admin status of any user except for the currently logged-in admin.

The matching route (found at the end of `/config/routes.rb`) is as follows:

```ruby
  patch '/users/:id/admin', to: 'users#admin_toggle', as: 'user_admin_toggle'
```

All I had to do after that was create a link or button to that route in my admin-only users view. If your application does not use jQuery, be sure to use `button_to` instead of `link_to` in your view code. 

```erb
<% @users.each do |user| %>
  <div class="user_link">
    <%= link_to user.full_name, user_path(user) %><br>
    <% if user.admin %>
      <%= link_to 'Revoke admin', user_admin_toggle_path(user), method: 'patch' %>
    <% else %>
      <%= link_to 'Set admin', user_admin_toggle_path(user), method: 'patch' %>
    <% end %>
      | <%= link_to 'Delete user', user_path(user), method: 'delete' %>
    <br>
    <br>
  </div>
<% end %>
```

There are tons of potential uses for this kind of functionality, especially when it comes to changing a boolean value in your models. Play around with it, but do be sure to protect your controller actions.
