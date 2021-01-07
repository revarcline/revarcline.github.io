[Rhymera](https://github.com/revarcline/rhymera) is a ruby CLI utility to search the [RhymeBrain](https://rhymebrain.com) API for rhymes and portmanteaus. Users are able to find additional details by selecting each word, such as the root words of the portmanteau and number of syllables in each rhyme, as well as copy any given result to the system clipboard. This was my first ruby gem, and the process of putting it together involved getting over a few hurdles.

One particularly fun piece was using [TTY::Prompt](https://github.com/piotrmurach/tty-prompt) to handle the menuing for the application. Skipping the tedious process of writing code to paginate out results myself, I could simply pass a hash of menu entries pointing to strings, objects, or even lambdas to process the results of each search.

Lambdas, in particular, were especially helpful. In the following method, part of the `Rhymera::Menu` class, I pass instance variables through lambdas to be able effectively use these entries at every level of the menu.

```ruby
    def extra_menu_entries
      # options leading to lambas! we love lambdas, don't we folks
      [{ "Copy '#{@word}' to Clipboard" => -> { Clipboard.copy(@word) } },
       { "Search '#{@word}'" => -> { search(@word) } },
       { 'New Search' => -> { call } },
       { 'Previous Searches' => -> { old_searches } },
       { 'Quit Program' => -> { exit(0) } }]
    end
```

With the lambda (`->`) operator, I am able to store the intended functionality as a block in each hash in the array. When it is returned by a `TTY::Prompt.select` call, the relevant block is executed. Easy as pie! The `tty-prompt` gem is a gift, I highly recommend playing with it.
