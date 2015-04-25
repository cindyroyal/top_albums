#Middleman Top Albums Example
#Created by Skip Baney of Vox Media for Texas State's Coding and Data Skills Course.

This will walk you through the process of building an editorial app using middleman.


## 0. Ensure your computer is configured correctly:

```
# Ensure there's a gem directory in your user folder.
mkdir -p "~/.gem"

# Set an environment variable to tell rubygems that you 
# want to install gems in your home directory (where you 
# have permission to write to).
export GEM_HOME="$HOME/.gem"

# Gems can install executables (like `bundle` and `middleman`).
# Rubygems puts those in a directory in your GEM_HOME. We're 
# adding that location to the PATH environment variable so that
# can run those executables without having to reference them by
# their fullly qualified path (i.e. we want to type 'middleman',
# not '~/.gem/bin/middleman').
export PATH="$HOME/.gem/bin:$PATH"
```

Please note that the two variables are only set for a single terminal session. If you open
a new terminal window then you will need to re-run the export commands. Alternately, you can
put the export commands at the bottom of '~/.bash_profile' and they will be run when opening
every terminal window.

```
# Finally, make sure that middleman and bundler are installed:
gem install bundler
gem install middleman
```

## 1. Setup the base middleman app

This was initially done via `middleman init top_albums`, but you should check out the example app repo and switch to the `part_1` branch:

```
git clone git@github.com:twelvelabs/top_albums.git
cd top_albums
git checkout -b part_1 origin/part_1
```

Startup the middleman web server:

```
middleman server
```

Now load up the app: http://0.0.0.0:4567. There shouldn't be much to look at here, we're just making sure that everything is working correctly. Let's test out middleman build to see how that works:

```
middleman build
```

This should output some info about static files being generated in the ./build directory. Congrats - everything is working!

## 2. Build out a basic layout

```
git checkout -b part_2 origin/part_2
```

https://github.com/twelvelabs/top_albums/compare/part_1...part_2

Here, we are modifying `./source/layouts/layout.erb` to include a header that will be on all pages. It includes a link to the main index page of our app, as well as twitter and facebook links.

```
<nav class="m-sticky-nav">
  <ul class="buttons links">
    <li><a href="index.html">Top Albums</a></li>
  </ul>
  <ul class="buttons social">
    <li><a class="icon-twitter" href="#"></a></li>
    <li><a class="icon-facebook" href="#"></a></li>
  </ul>
</nav>
```

In addition, we're changing the main CSS file (`./source/stylesheets/all.css`) and adding a `.scss` extension. This is a feature of middleman that allows us to use the Sass CSS extension. Sass gives
us the ability to use CSS partials and variables, as well as a ton of other features (see: http://sass-lang.com/guide for more). Sass isn't _necessary_, but helpful to allow
us to reuse Vox Media CSS styles (because even demos should look pretty) - which are pulled in as Sass partials.


## 3. Stub out some pages

```
git checkout -b part_3 origin/part_3
```

https://github.com/twelvelabs/top_albums/compare/part_2...part_3

Let's add some actual pages for our app. We'll replace the contents of `./source/index.html.erb` with:

```
<div class="m-grid">
  <div class="row">
    <div class="col-md-3">
      <ul class="usernames">
        <li><a href="twelvelabs.html">twelvelabs</a></li>
        <li><a href="clockwerks.html">clockwerks</a></li>
      </ul>
    </div>
    <div class="col-md-9">
      <p>Lorem ipsum dolor...</p>
    </div>
  </div>
</div>
```

And add two files for the lastfm users: `./source/twelvelabs.html.erb` and `./source/clockwerks.html.erb`.


## 4. Grab top albums JSON from the lastfm API

```
git checkout -b part_4 origin/part_4
```

https://github.com/twelvelabs/top_albums/compare/part_3...part_4

Make two requests to the last.fm API for the [top albums](http://www.last.fm/api/show/user.getTopAlbums) for Skip (twelvelabs) and Trei (clockwerks). I'm using a personal API key which the class is free to use tonight, you can [create you own here](http://www.last.fm/api/account/create). I'm also using the curl command line utility to save the JSON to files in the `./data` directory. The `-o` flag tells it to save the output of the URL to a specific filename:

```
mkdir -p ./data
curl "http://ws.audioscrobbler.com/2.0/?method=user.gettopalbums&user=twelvelabs&api_key=197c50f4bfebf651ad22b922e270ee00&format=json" -o ./data/twelvelabs.json
curl "http://ws.audioscrobbler.com/2.0/?method=user.gettopalbums&user=clockwerks&api_key=197c50f4bfebf651ad22b922e270ee00&format=json" -o ./data/clockwerks.json
```

Note: the JSON is returned by the API in a single line which is pretty hard to read. I've passed it through [a JSON validator](http://jsonlint.com) that reformats it into something human readable.


## 5. Use the JSON data in our username pages

```
git checkout -b part_5 origin/part_5
```

https://github.com/twelvelabs/top_albums/compare/part_4...part_5

Middleman gives us the ability to access JSON or YAML files in the data directory as hashes in Ruby. We're going to use the album info in each JSON file to render out album images. Since the only difference in these two pages will be the last.fm username, we'll create a shared partial and use it in both files. This way we only have to make code changes in one place if we decide later to change how the albums should be marked up.

Create a file at `./source/_albums.html.erb`. Note the leading underscore. This tells middleman that this is a shared partial, and not a file that should be rendered as a HTML page. Add the following to that file:

```erb
<div class="m-grid">
  <div class="row">
    <div class="col-md-3">
      <ul class="usernames">
        <li><a href="twelvelabs.html" class="<%= 'active' if username == 'twelvelabs' %>">twelvelabs</a></li>
        <li><a href="clockwerks.html" class="<%= 'active' if username == 'clockwerks' %>">clockwerks</a></li>
      </ul>
    </div>
    <div class="col-md-9">

      <% data[username]['topalbums']['album'].each do |a| %>
        <a href="/<%= username %>/<%= a['@attr']['rank'] %>.html"><img src="<%= a['image'][1]['#text'] %>" width="64" height="64" /></a>
      <% end %>

    </div>
  </div>
</div>
```

This is similar to the landing page, but instead of rendering out lorem ipsum text it's looping over the JSON data for each user. Next, we'll use this partial in `./source/twelvelabs.html.erb` and `./source/clockwerks.html.erb`, passing in the correct username for each file:

```
<%= partial('albums', locals: { username: 'twelvelabs' }) %>
```

and

```
<%= partial('albums', locals: { username: 'clockwerks' }) %>
```

When you load http://0.0.0.0:4567/twelvelabs.html, you should see a list of albums images. Note that they link to a non-existent set of pages. We'll be creating those next.


## 6. Create dynamic pages for each album

```
git checkout -b part_6 origin/part_6
```

https://github.com/twelvelabs/top_albums/compare/part_5...part_6

Create a file at `./source/detail.html.erb`. This will be used for showing each album. Add the following to it:

```erb
<div class="m-grid">
  <div class="row">
    <div class="col-md-3">
      <ul class="usernames">
        <li><a href="twelvelabs.html" class="<%= 'active' if username == 'twelvelabs' %>">twelvelabs</a></li>
        <li><a href="clockwerks.html" class="<%= 'active' if username == 'clockwerks' %>">clockwerks</a></li>
      </ul>
    </div>
    <div class="col-md-9">
      <p>
        <a href="<%= album['url'] %>"><img src="<%= album['image'][3]['#text'] %>" width="300" height="300" /></a>
      </p>
      <p>
        <a href="<%= album['artist']['url'] %>"><%= album['artist']['name'] %></a> -
        <a href="<%= album['url'] %>"><%= album['name'] %></a>
      </p>
      <p><%= album['playcount'] %> total plays</p>
    </div>
  </div>
</div>
```

This page assumes that the `username` and `album` variables will be passed into it. We'll wire those up next. Add the following to `./config.rb`:

```
data['clockwerks']['topalbums']['album'].each do |a|
  proxy "/clockwerks/#{a['@attr']['rank']}.html", 'detail.html', locals: { username: 'clockwerks', album: a }, ignore: true
end
data['twelvelabs']['topalbums']['album'].each do |a|
  proxy "/twelvelabs/#{a['@attr']['rank']}.html", 'detail.html', locals: { username: 'twelvelabs', album: a }, ignore: true
end

```

Also, to make the relative references work above, make sure to remove the comment for that section in the config.rb file.
 # Use relative URLs
  activate :relative_assets
