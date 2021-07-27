Using environment variables to store developer secrets has always been tricky for me. I've used things like [dotenv](https://github.com/bkeepers/dotenv) but as I moved into Elixir/Phoenix development, I was having a hard time finding a similar solution. Enter [direnv](https://direnv.net/), a far more elegant solution that won't pollute your `.profile` or your project modules.
<!--more-->

To install direnv, first install the relevant package for your os, direnv's site offers helpful instructions [here](https://direnv.net/docs/installation.html). After it's installed, time to configure your shell. I use `zsh`, so I added the following line at the end of my `.zshrc`:


```bash
# at bottom of .zshrc
eval "$(direnv hook zsh)"
```

For bash, replace `zsh` with `bash`. Now, be sure to restart your shell or source the `.zshrc` so the direnv hook is loaded.

Now it's time to get started. The beauty of direnv is the way it's completely project-agnostic, so this will work for nearly any project you have going. At your project root, add the following to your `.gitignore` so you don't upload your secrets to the public internet:

```
# hide environment variables with dev secrets
.envrc
```

Now, time to get going. Create an `.envrc` file like so:

```bash
export MY_SECRET="supersecret"
export MY_OTHER_SECRET="supersecret"
```

Note that there cannot be any spaces in the variable assignment. You should see a message like the following once you return to your shell.

```
direnv: error /home/arcline/code/gh-pages/_posts/_drafts/.envrc is blocked. Run `direnv allow` to approve its content
```

Well, that's a great sign! That's a sure sign `direnv` is loaded in your shell, but isn't allowed to get access to your new `.envrc` file. Go ahead and execute `direnv allow .envrc` and your shell should load your secret environment variables. Now whenever you enter the directory, those enviroment variables will be loaded in the shell.

Most languages have an interface to call variables from the system. Using the example environment variables above, let's show how some of the more common ones work.

Ruby:
```ruby
ENV["MY_SECRET"]
```

Node.js:
```javascript
process.env.MY_SECRET
```

Elixir:
```elixir
System.get_env("MY_SECRET")
```

Python:
```python
import os
os.environ.get("MY_SECRET")
```

There you go! No more fiddling with a different system for each language, now you can hide API keys and secrets securely with ease.
