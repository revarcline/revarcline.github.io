Building the [craziest-eights](https://craziest-eights.herokuapp.com/) API for [a hackathon](https://mintbean.io/meets/7e2331fb-1e0d-4b31-86b9-a46acad877af) involved a number of fun techincal challenges, but a big one was keeping the database clean. The model was set up specifically to only persist for the duration of the game, so I needed to find a way to clear games out of the database on my deployed instance. Fortunately, heroku offers a convenient [scheduler add-on](https://devcenter.heroku.com/articles/scheduler) that allowed me to run a daily rake task without having to dive into the crontabs or create a special service.
<!--more-->

My specific need was to clear out any games that have been left idle for more than six hours on a daily basis. First, I added a check to my `Game` and `Deck` models:

```ruby
def self.pending_deletion
  where('updated_at < ?', 6.hours.ago)
end
```

Next, I created a file at `/lib/tasks/scheduler.rake`:
```ruby
desc 'clear idle games'
task clear_idle: :environment do
  puts 'clearing old games...'
  Deck.pending_deletion.destroy_all
  Game.pending_deletion.destroy_all
  puts 'idle games cleared.'
end
```

Now I had a rake action to clear any games (and decks) that were no longer in use.

Scheduling it with heroku was simple, I just had to follow the documentation (linked above). From my project root, I ran:
```
$ heroku addons:create scheduler:standard
```

This allocated the scheduler to my project, and `heroku addons:open scheduler` opened up heroku's scheduler interface for me. I just had to click the Add Job button, and have it run the command `rake clear_idle` at 00:00 UTC daily. Easy as pie!
