My most recent project was a small sinatra-based emulator ROM libarary called [rombrary](https://github.com/revarcline/rombrary). Working up to the project, one of the concepts the flatiron lesson materials introduced was the idea of a 'slug', or a url-friendly version of a resource's name, usually generated by stripping all non-alphanumeric characters, setting the string to downcase, and replacing whitespace with hyphens. For instance, "Super Mario World" would become "super-mario-world".
<!--more-->

Now, the way we first implemented this in our lessons was to create a module with methods that our different models could extend/include. ActiveRecord is fun, though. Instead of creating a `slug(name)` method and having to call that every time we wanted to make a slug for a resource, we can actually do some fun metaprogramming to the `name=(name)` method! Here's an example from the `slugable.rb` [file](https://github.com/revarcline/rombrary/blob/main/app/models/concerns/slugable.rb) in the rombrary project.

```ruby
module Slugifiable
  def make_slug(name)
    name.downcase.gsub(/[^a-z0-9\s]/, '').gsub(' ', '-')
  end

  def name=(name)
    self.slug = make_slug(name)
    super
  end

  # for user model
  def username=(name)
    self.slug = make_slug(name)
    super
  end
end
```

All that I had to do to make each model automatically generate slugs was to `include Slugifiable` and make sure the accompanying migration included a `slug` column, and ActiveRecord would take care of the rest.

I'm definitely looking forward to seeing more automagical stuff like this happen when we move on to Rails.