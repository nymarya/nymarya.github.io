---
layout: post
title: "Self-updating portfolio on README"
categories:
  - eng
tags:
  - python
  - github actions
  - readme
---

At july 10h,  2020, I was happily navigating through my Twitter timeline when this tweet popped to my eyes:

<div>
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Made myself a self-updating GitHub personal README! It uses a GitHub Action to update itself with my latest GitHub releases, blog entries and TILs <a href="https://t.co/Eve7FOrwYK">https://t.co/Eve7FOrwYK</a> <a href="https://t.co/oJPXLtFdgM">pic.twitter.com/oJPXLtFdgM</a></p>&mdash; Simon Willison (@simonw) <a href="https://twitter.com/simonw/status/1281435464474324993?ref_src=twsrc%5Etfw">July 10, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

To be honest, a developer's existency is already described in a GitHub profile, the [about](https://nymarya.github.io) page of a personal website, the LinkedIn description, sometime even a Twitter bio, and the automatized approach was the thing that really made me want to build a README on GitHub. After seeing Simon's ideia, the new feature looked more interesting, offering a opportunity to show sintetized informations that do not require manual updating (and therefore decreases the risk of not being updated at all), quality that the other options do not present.

### Starting a automazed README

The first step is to choose which informations should appear. In my case, I wanted to show my posts and a overview of the tools used in my Github repositories.

The auto-desciption itself I choose to write manually because: I wanted to use emojis; to write posts both in english and portugues; some personal information, such as hobbies, I couldn't retrieve from anywhere; and, yes, I coul create a script to simply append the text that I would manually choose, but I don't think it would look pretty and tidy.

With that in mind, it's time to start.

### Adding posts with BeautifulSoup

In his [post](https://simonwillison.net/2020/Jul/10/self-updating-profile-readme/), Simon retrieve his posts in a more elegant way, but unfortunately my Github Pages blog doesn't offer me the same tools. However, it has a HTML strutuctred that is very well-formed, so it wasn't so hard to get what I wanted from it.

The [BeautifulSoup](https://pypi.org/project/beautifulsoup4/) is great for mining information from websites. To get the posts divided by each language, I got the HTML of the [page](https://nymarya.github.io/categories) where the posts are categorized, identified where (which tags) was located each category, and simply saved recovered the address, title and date of a number (previously chosen) of posts. The full code is  available [here](https://github.com/nymarya/nymarya/blob/master/posts.py).

### Adding profile overview

Once again, Simon's solution is somewhat more sofisticated. This time, he used the Github's GraphQL API in order to retrieve data from GitHub in a way that calls way less HTTP requests.

I decidesd to use REST API for practicity reasons. I used is twice: once to get the names of the repositories (`https://api.github.com/users/<usuario>/repo`) and once to get the languages used in each repository (`https://api.github.com/repos/<usuario>/<repo>/languages'`).The unantenticated version of the Github API, besides providing access only for public repositories, has a limit of 60 requests per hour, so if you have 60 repositories or more, you should use at least the authenticated version. All requests are made with the self-named library, `requests`.

The response for the second API consists of a dictionary containing each laguage and its byte count. Then, to get the proportion is only needed to append all information and finally calculate the percentages. To show this information, I used a tabel that contains the logos in the first line (a great number of those come from this [repository](https://github.com/abranhe/programming-languages-logos) and the second line shows the name and percentage of code in which each language is use. The code is available [here](https://github.com/nymarya/nymarya/blob/master/repositories.py).

### Finally, the self-updating part

All writing process is defined in the [build_readme.py](https://github.com/nymarya/nymarya/blob/master/build_readme.py) file. But in order to insert text in the file, first it is necessary to  indicate in the README.md where the information will be placed. This is done by making comments in the mardown that will mark the sections:

```markdown
<!-- <nome da seção> starts -->
<!-- <nome da seção> ends -->
```

Then, using the `re` library we can use regular expressions to identify these blocks. The expression used `"<!-- <nome da seção> starts -->.*<!-- <nome da seção> ends -->"`. The `.*` serve as a way to capture all text inside the section, so in the next update all previous contant will be replaced with the new one. When inserting the text per se, it's important that the comments remain around the new content, because that way the block can be found when the script runs again.

The final step is to set up the workflow with Github Actions. This is done by going to the tab, choosing "set up a workflow yourself" and then adding the commands to set up python, configure pip, install dependencies from requeirements.txt, update README.md and finally commiting the result, besides setting the cron so the task will run every hour. The workflow is almost the same as Simon's, the only difference is that his has one more command to get enviromental variables.

### For those who have time and energy

There is still a bunch of stuff that can de automated. The most recent releses (as Simon did), counting the commits, questions and ansers from StackOverflow (I don't know if they have a API, if don't certainly BeautifulSoup is there for you), current job and education info on LinkedIn (again API or BeautifulSoup must do the job), publications on LinkedIn, posts on Medium, most recents videos or podcasts , and so on. Creativity is welcome.

I'm ansious to see the next ideias. See ya!

## Links

[Simon's post where he explains it all](https://simonwillison.net/2020/Jul/10/self-updating-profile-readme/)

[Yet another Simon's post on README automation](https://simonwillison.net/2020/Apr/20/self-rewriting-readme/)
