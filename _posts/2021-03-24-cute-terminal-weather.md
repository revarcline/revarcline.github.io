I'm a big fan of Igor Chubin's [wttr.in](https://wttr.in/) as a lovely little tool to look up the weather in the terminal. You even get color ascii art!

Here's an example of it in action:

![current weather](/assets/images/wttr.png)

<!--more-->

To make using it a little easier in the terminal, I added the following to my `.zshrc` (this should also work in your `.bashrc`):

```bash
# weather forecast (2 column)
wf ()
{
    local city="${1:-New Orleans}"
    curl wttr.in/$city\?nqF
}
# current weather
cw ()
{
    local city="${1:-New Orleans}"
    curl wttr.in/$city\?0qF
}
```

Simply replace `New Orleans` with your own city. Of note, if you are using it to look up another city with a two word name, note that the space will end the argument unless you denote it as literal. For instance, run it as `cw NewYork`, `cw New\ York`, `cw 'New York'`, or even `cw NewYork` will work. Zip codes should work too.
