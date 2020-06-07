+++
title = "Using Tabix in Windows WSL"
subtitle = "All on a Dell G7:rocket:"

date = 2018-12-22T00:00:00
lastmod = 2018-12-22T00:00:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ['Fausto Lopez']

tags = ["Academic"]
summary = "A quick guide connecting your local clickhouse database to Tabix for business intelligence."

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["deep-learning"]` references 
#   `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
# projects = ["internal-project"]

# Featured image
# To use, add an image named `featured.jpg/png` to your project's folder. 
[image]
  # Caption (optional)
  caption = ""

  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = ""

  # Show image only in page previews?
  preview_only = false

# Set captions for image gallery.
+++

## What is Tabix?

As part of my clickhouse setup series, I wanted to have a gui to work with, especally as a way for analysts without R and Python proficiency to query the database. [Tabix](https://tabix.io/) seemed perfect for this; in addition the software had dashboarding capabilities for automated reports which seemed perfect. 

##Connecting

Setting up tabix turns out to be incredibly simple. If you have followed my previous posts and set up clickhouse under wsl, then clickhouse will be running off localhost. First make sure cickhouse is running in wsl:

![optional caption text](/img/clickhouse_tabix_intro/start_clickhouse.png)


Since tabix can run off your browser it doesn't even need to be installed. Simply launch the (browser)[http://ui.tabix.io/] and you will be prompted to connect:

![optional caption text](/img/clickhouse_tabix_intro/connect.png)


Under name simply set up a name for your connection, in my case I called it local_ch, then point tabix to local host port 8123. Note that if you are setting up remotely this will be different. I plan to do a tutorial on remote setup later. Under login you'll want to type 'default' since in my set up I haven't added security; once again this will change if we set up remote systems and security in clickhouse. Once you are set, click sign in (I chose to click add new as well in order to store the data).

#Querying

Once you sign in you will see the querying screen which is pretty intutive. Try and run the first query we ran from [Mark Litwintschik's blog](https://tech.marksblogg.com/). You should receive the following error:

![optional caption text](/img/clickhouse_tabix_intro/error.png)

To my knowledge this occurs becuse query time is greater than 5 seconds which is the default query time. Note that query time will vary based on your specs and tuning. In my case I didn't tune the databse yet, and I also didn't create the mergetree version of the table Mark did yet (you will see he creates a smaller table to query with optimized fields in his blog). Regardless I wanted to pump up the query time; all you have to do is go to settings on the top right and change the max exexution time in the new window from 5 to what your want (I put in 100 seconds):

![optional caption text](/img/clickhouse_tabix_intro/fix.png)

Click apply on the bottom and you should be good to go. Rerun the query and voila:

![optional caption text](/img/clickhouse_tabix_intro/query.png)

Depending on the data you loaded you'll have different numbers, note you can use the x circled in red to save your table.

##Conclusions
And that's how easy it is to connect tabix to your clickhouse instance locally under wsl. In my next post in this series I'll explore how to set up some quick graphs for dashboard reports, something that is very valuable when dealing with large data like this.