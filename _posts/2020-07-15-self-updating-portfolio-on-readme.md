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

At July 10h, 2020, I was happily navigating through my Twitter timeline when this tweet popped to my eyes:

<div>
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Made myself a self-updating GitHub personal README! It uses a GitHub Action to update itself with my latest GitHub releases, blog entries and TILs <a href="https://t.co/Eve7FOrwYK">https://t.co/Eve7FOrwYK</a> <a href="https://t.co/oJPXLtFdgM">pic.twitter.com/oJPXLtFdgM</a></p>— Simon Willison (@simonw) <a href="https://twitter.com/simonw/status/1281435464474324993?ref_src=twsrc%5Etfw">July 10, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

A developer's existence is already described in a GitHub profile, the [about](https://nymarya.github.io) page of a personal website, the LinkedIn description, sometimes even a Twitter bio, and the automated approach was the thing that made me want to build a README on GitHub. After seeing Simon's idea, the new feature looked more interesting, offering an opportunity to show synthesized information that doesn't require manual updating (and therefore decreases the risk of not being updated at all), a quality that the other options don't present.

### Starting an automated README

The first step is to choose which informations should appear. In my case, I wanted to show my posts and an overview of the tools used in my Github repositories.

The auto-description itself I choose to write manually because: I wanted to use emojis; to write posts both in English and Portuguese; some personal information, such as hobbies, I couldn't retrieve from anywhere; and, yes, I could create a script to simply append the text that I would manually choose, but I don't think it would look pretty and tidy.

With that in mind, it's time to start.

### Adding posts with BeautifulSoup

In his [post](https://simonwillison.net/2020/Jul/10/self-updating-profile-readme/), Simon retrieves his posts in more elegantly, but my Github Pages blog doesn't offer me the same tools. However, it has an HTML structured that is very well-formed, so it was easy to get what I wanted from it.

The [BeautifulSoup](https://pypi.org/project/beautifulsoup4/) library is great for mining information from websites. To get the posts divided by each language, I got the HTML of the [page](https://nymarya.github.io/categories) where the posts are categorized, identified where (meaning: which tag) was located each category, and simply recovered the address, title, and date of a number (previously chosen) of posts. The full code is available [here](https://github.com/nymarya/nymarya/blob/master/posts.py).

### Adding profile overview

Once again, Simon's solution is somewhat more sophisticated. This time, he used the Github's GraphQL API to retrieve data from GitHub in a way that calls way fewer HTTP requests.

I decided to use the REST API for practical reasons. I used it twice: once to get the names of the repositories (`https://api.github.com/users/<usuario>/repo`) and once to get the languages used in each repository (`https://api.github.com/repos/<usuario>/<repo>/languages'`). The unanthenticated version of the Github API, besides providing access only for public repositories, has a limit of 60 requests per hour, so if you have 60 repositories or more, you should use at least the authenticated version. All requests are made with the self-named library, `requests`.

The response for the second API consists of a dictionary containing each language and its byte count. Then, to get the proportion is only needed to append all information and finally calculate the percentages. To show this information, I used a table that contains the logos in the first line (a great number of those come from this [repository](https://github.com/abranhe/programming-languages-logos)) and the second line shows the name and percentage of code in which each language is used. The code is available [here](https://github.com/nymarya/nymarya/blob/master/repositories.py).

### Finally, the self-updating part

All writing process is defined in the [build_readme.py](https://github.com/nymarya/nymarya/blob/master/build_readme.py) file. But to insert text in the file, first, it is necessary to indicate in the README.md where the information will be placed. This is done by making comments in the markdown that will mark the sections:

```markdown
<!-- <section name> starts -->
<!-- <section name> ends -->
```

Then, using the `re` library we can use regular expressions to identify these blocks. The expression used `"<!-- <section name> starts -->.*<!-- <section name> ends -->"`. The `.*` characters is used to capture all text inside the section, so in the next update, all previous content will be replaced with the new one. When inserting the text per se, the comments must remain around the new content, because that way the block can be found when the script runs again.

The final step is to set up the workflow with Github Actions. This is done by going to the tab, choosing "set up a workflow yourself" and then adding the commands to set up python, configure pip, install dependencies from requeirements.txt, update README.md and finally committing the result, besides setting the cron so the task will run every hour. When you save the `main.yml` containing the configurations in the directory `.github/workflows`, o GitHub Actions will identify the file right away. The workflow is almost the same as Simon's, the only difference is that his setup has one more command to get environmental variables.

### For those who have time and energy

There is still a bunch of stuff that can be automated. The most recent releases (as Simon did), counting the commits, questions and answers from StackOverflow (I don't know if they have an API, if don't certainly BeautifulSoup is there for you), current job and education info on LinkedIn (again API or BeautifulSoup must do the job), publications on LinkedIn, posts on Medium, most recent videos or podcasts, and so on. Creativity is welcome.

I'm anxious to see the next ideas. See ya!

## Links

[Simon's post where he explains it all](https://simonwillison.net/2020/Jul/10/self-updating-profile-readme/)

[Yet another Simon's post on README automation](https://simonwillison.net/2020/Apr/20/self-rewriting-readme/)
