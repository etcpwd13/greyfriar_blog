---
title: "Signed up for Hack The Box"
date: 2020-12-28T15:34:30-04:00
categories:
  - blog
tags:
  - hackTheBox
  - update
---

I signed up for Hack the Box last night. In order to get an invite code from them youfirst have to hack the site [Index page][htb-index]. to do this you have to do several things to get an invite code from the site API.

You can follow the video for how to do it [here][howto-video]
1. Open [Developers console] [dev-console]

2. Find and run the suspicious java script by making a cop of it and using the script name() in console to run it. you will get some encoded string that needs decoding

3. Go to this [Decode site][decode-site]

4.Finally the decoded message ask you to submity a POST request to the site API. use online [Post site][post-site] to do this!

Once I was signed up I had to download some VM and VPN software to access the lab network

*How To Post in this BLOG*

You'll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.




[htb-index]: https://www.hackthebox.eu/home
[dev-console]: https://developers.google.com/web/tools/chrome-devtools/open#:~:text=Press%20Command%20%2B%20Option%20%2B%20J%20(,Get%20Started%20With%20The%20Console.
[howto-video]: https://codeburst.io/hack-the-box-how-to-get-invite-code-56e369fc8dae
[decode-site]: https://soumya.dev/decode
[post-site]: https://reqbin.com/
