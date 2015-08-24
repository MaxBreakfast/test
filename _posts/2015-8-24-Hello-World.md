---
layout: post
title: Creating a dynamically updating search field using ui-select!
---

I was recently working on project utilizing the Spotify api in order to search spotify’s library and select a song. For this project, my team and I used the MEAN stack. Obviously, in order for the user to search Spotify’s library, we needed to have a search field on the front end. However, angular does not have any native means to create a dropdown menu that updates with suggested options. This creates a problem since the user never receives feedback from the search field and therefore is cannot be sure whether their search is actually returning valid data. Additionally if the user is not getting a list of suggested options then they are required to fully type their desired option in order to select a song (which is inconvenient).

After doing some research, I discovered an angular plugin called ui-select. ui-select abstracts away the code needed to build a functional search drop-down.  




The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.
