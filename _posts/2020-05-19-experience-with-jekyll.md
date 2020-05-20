---
layout: single
classes: wide
title: Experience with Jekyll.
date: 2020-05-19 19:37 -0500
author_profile: true
read_time: true
comments: false
share: true
related: false
---
The days following my Ph.D. defense and dissertation submission was the perfect time for me
to create a personal website, something I had been waiting to do for a long time.
I was able to create a website in slightly over a weekend with [Jekyll](https://jekyllrb.com/).
Jekyll is an awesome tool to create static websites, like a personal website or a portfolio.
Another popular tool I came across was [Hugo](https://gohugo.io/). However, I
decided to stick with Jekyll somehow since I had heard about it earlier. An advantage of
using Jekyll is that you get many options for templates to choose from. In my case, I would
like to thank [Michael Rose](https://mademistakes.com/) for the great looking, feature packed
[Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/) Jekyll template. The
sites are also mobile-friendly. I should also mention that you can also host your Jekyll
website for free via [Github Pages](https://pages.github.com/) or
[Netlify](https://www.netlify.com/). No ads!

The Jekyll docs and online resources are great places to begin. But, I just wanted to
mention a few things that might make things a little easier (at least it did for me).
Also, what I mention here holds in general and can be considered a *good practice*.
Feel free to adopt the principle before you start working on your personal website
with Jekyll, or some other tool.

## Check if running a container works for you
If you have installation issues with either Jekyll or its dependencies (which is in
[Ruby](https://www.ruby-lang.org/en/)), try the jekyll [docker container](https://hub.docker.com/r/jekyll/jekyll/).
[Docker](https://www.docker.com/) has become popular even in scientific computing
and analysis (apart from the industry), specially astrophysics. Also, if you are not
an active web developer, but just want Jekyll for only your website, this is the
cleanest option in my opinion.

I did the following:
- I had a dedicated terminal for the container. After you have installed `docker`
  and pulled down the container using `docker pull jekyll/jekyll`, run
  the following in this terminal.
  ```shell
  docker run -it  --publish 4000 \
      --volume /home/deep/github/:/static-site/ \
      --name my-jekyll-container \
      jekyll/jekyll bash
  ```
  Let me break down the options starting from the bottom.
  - The `jekyll/jekyll` is the Jekyll image you pulled down from Dockerhub. In that
    container, you will run `bash`.
  - Give this container a name. I chose `my-jekyll-container`. You can refer
    back later to this container using this name.
  - The `--volume` does something important. Turns out that a container is like an
    independent OS running on your system with its own filesystem. When you exit
    a container it removes files you create, unless you make it persistent. Here,
    I'm linking my local machine's `/home/deep/github/` folder to the `/static-site/`
    folder of the container. Below is the output of `ls` in the started `bash` process.
    ```shell
    bash-5.0# ls
    bin          home         mnt          root         srv          tmp
    dev          lib          opt          run          static-site  usr
    etc          media        proc         sbin         sys          var
    ```
    There is a `static-site`, the contents of which will be identical to
    `/home/deep/github/` of my laptop or local machine. Files created under
    `static-site` will be visible outside even after the container is stopped.
  - The `--publish` is used to expose ports of the container outside it. Here, I'm
    exposing port `4000` since the jekyll server runs on `localhost:4000`. I'll be
    saying more on this later down.
  - The `-it`  stands for **i**nteractive **t**erminal. It is needed since you will
    be entering commands in the `bash` process you just started in the container.

- Now, I opened a different terminal window (not running `docker`) and navigated
  to my `/home/deep/github` and initiated a `git` repository which will host my
  static site.

- My intention being that I will run `jekyll` commands from the first terminal to
  build and locally *see* the website. But for everything else, like editing the
  files, or checking out the files to `git`, I will use the second terminal (or
  for that matter as many new terminal tabs I require). **Bottomline**: One terminal
  running Jekyll container to build and serve the website. Other terminal windows
  to do eveything else.

- Now back to the `--publish`. Inside of the container you will frequently run
  `bundle exec jekyll serve` which starts the jekyll server for you to see the
  website in action. The address is typically `localhost:4000`. If you were not
  running a container, you will be able to preview the website if you entered this
  address in a web browser. However in this case, the `localhost` refers
  to the container and *its* port `4000`. This is why we exposed it using `--publish`.
  You need to find out which port outside the container it corresponds to. So
  in the second terminal (not running `docker`) I ran the following,
  ```shell
  docker port my-jekyll-container 
  4000/tcp -> 0.0.0.0:32769
  ```
  which says container port `4000` corresponds to host `0.0.0.0`, port `32769`
  (the number may be different in your case). So you need to supply this information
  to jekyll when running the server. I did the following to start the server inside
  the container,
  ```shell
  bundle exec jekyll serve --host 0.0.0.0
  ```
  and then found the preview in a browser when I entered `localhost:32769`. That's
  it. So to recap, the terminal with the  container runs the jekyll server, you
  make edits and check out files to `git` using other terminal windows, and see the
  preview on your browser.

## Some images are rotated

Another issue I found when adding images is that some of photos I had taken on
my phone always showed up rotated on the site preview. After some digging around,
I eventually found the issue to be what the orientation of the phone was when
I took the photo (which, of course, you cannot do anything about now). Apparently,
the meta information of the photo was jumbled up as to what was *top* or *bottom*.
Fortunately, there was a simple fix.

- Firstly you need to check if this is the case. Some more digging pointed to
  [Exiftool](https://exiftool.org/) which is available in package managers.
  I installed it and ran the following on the misbehaving images.
  ```shell
  exiftool -Orientation my-image.jpg
  Orientation                     : Rotate 90 CW
  ```
  This confirmed that indeed this was the case.

- For me, I tried to open the picture in GIMP which comes pre-installed in
  Debian systems, and it asked me rightaway, if I wanted to fix the orientation.
  All I had to do is to export it with a different filename from GIMP.
  Later, I came to know that this also happens with tools like Photoshop.
  It was a serendipitous, much welcome fix!


I should mention that the [LinkedIn course](https://www.linkedin.com/learning-login/share?forceAccount=false&redirect=https%3A%2F%2Fwww.linkedin.com%2Flearning%2Flearning-static-site-building-with-jekyll%3Ftrk%3Dshare_ent_url&account=77313426) by Nate Barbettini helped me get a quick start
with Jekyll. Otherwise, if you decide to use the same template as I did, the
[quick start](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)
should be good enough. Happy Jekylling!
