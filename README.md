# Flying Rat Blog

This is tech blog of Flying Rat development team built on amazing [Jekyll](https://jekyllrb.com) site generator (including nice Minimal Mistakes theme!) and hosted by [Github Pages](https://pages.github.com) :raised_hands:

## How To Create a Post

Just create a new `.md` file and place it to `/_posts/` folder. Then fill required meta information like `date` or `title` and - of course - your amazing blog post text! :sunglasses:

After merging PR to `main` branch - Github Pages will automatically build and deploy new version of the blog :rocket:

# Development

As this blog is based on Jekyll - various parts of it can be modified and changed, feel free to make changes and submit PR!

## Requirements

Following tools needs to be installed on your development machine:

- Ruby
  - Tested with Ruby 3.2.\* on Windows WSL installed using [asdf](https://asdf-vm.com/)
    
> [!WARNING]
> Do not use Ruby 3.3.\* as it's not supported by Github Pages yet

## Start Local Instance

If you want to start local environment just use following:

```shell
bundle install # Install all dependencies see Gemfile for more
bundle exec jekyll serve --livereload # Starts local HTTP server with live reload enabled
```
