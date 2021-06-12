
In the course of creating [cellarmonster](https://cellarmonster.buckar.ooo/), an inventory/point of sale system for wine in restaurants, one of the first features I wanted to make sure I implemented was a robust search system. Thankfully, this is incredibly easy in rails with the [pg_search](https://github.com/Casecommons/pg_search) gem. By leveraging the full text search feature of postgres and integrating it with ActiveRecord, you'll have a powerful (and fast!) tool to search the strings in any record of your rails project.
<!--more-->

First, installl it in your rails project by adding `gem 'pg_search'` to your `Gemfile`. After running `bundle install`. To set up multi-search (text search over multiple resources), first run the following in your project root directory:

```
rails g pg_search:migration:multisearch
rails db:migrate
```

These commands will create the necessary migrations for `pg_search` to index the text of all your records. To implement the functionality, you need to call it in your models. Here's an example from cellarmonster:

```ruby
class Bottle < ApplicationRecord
  include PgSearch::Model
  multisearchable against: %i[name appellation color price vintage notes sku region format price]
  # rest of model code
end

class Country < ApplicationRecord
  include PgSearch::Model
  multisearchable against: [:name]
  # rest of model code
end
```

`pg_search` is incredibly robust and offers a number of ways to delimit the paramaters of multisearch, the documentation offers plenty of examples.

After adding the functionality to all my relevant models, I implemented a search controller. Of note, I wanted got a list of unique results, so I added a helper method to flatten it out to just bottles. I also make sure to add the search query to my json object so my frontend can display it.

```ruby
class SearchController < ApplicationController
  def index
    results = flat_results(PgSearch.multisearch(params[:query]).map(&:searchable))
    result_hash = BottleSerializer.new(results).serializable_hash
    result_hash[:resource_name] = params[:query]
    render json: result_hash.to_json
  end

  protected

  def flat_results(results)
    # return bottle records when search returns other resource
    # somewhat inefficient but works fine at this scale for now
    all_bottles = results.map do |record|
      if record.class.to_s == 'Bottle'
        record
      else
        record.bottles
      end
    end

    all_bottles.flatten.uniq
  end
end
```

From there, it's a simple matter of adding the following line to `routes.rb`:

```ruby
Rails.application.routes.draw do
  # ...
  get 'search/:query', to: 'search#index'
  # ...
end
```

To use this route on the frontend, I created a `Search` component i would insert into my navigation bar:

```javascript
import React, { useState } from 'react';
import { Form, Button, InputGroup } from 'react-bootstrap';
import { useHistory, withRouter } from 'react-router-dom';

const Search = () => {
  const history = useHistory();
  const [query, setQuery] = useState('');

  const onSubmit = (event) => {
    event.preventDefault();
    history.push(`/search/${query}`);
  };

  return (
    <Form onSubmit={onSubmit} inline>
      <InputGroup>
        <Form.Control
          type="text"
          value={query}
          onChange={(event) => setQuery(event.target.value)}
          placeholder="Enter a term or SKU"
        />
        <InputGroup.Append>
          <Button type="submit">Search</Button>
        </InputGroup.Append>
      </InputGroup>
    </Form>
  );
};

export default withRouter(Search);
```

The `/search/:query` route in my router sends the user to the `BottleList` component, which dispatches the call to the API and displays the relevant records.

![bottle search in action](/assets/images/bottle_search.png)

You can use the search bar to return results for any bit of text associated with the record, and `pg_search` does all the hard work for you!
