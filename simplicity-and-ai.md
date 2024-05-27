# Simplicity and AI

## The Problem
I've recently wanted to update my [personal site](https://benjaminfleckenstein.name) 
and thought about what tool to use. I wanted a simple site without fuzz, good readability both on desktop and mobile and
without a huge software stack. After all it's mostly technical articles, I don't want to run a container stack just for a few
HTML pages. I tried Jekyll just for the fun of it, but it was relatively cumbersome, so I
returned to my own [sitegenerator project](https://github.com/nbkr/sitegenerator). The sitegenerator is a small
tool that allows you to write Markdown articles in their own Git repositories
and combine them into a blog - essentially, a miniature release management
system.

## The solution and the way there
However, I quickly realized that I didn't really need such a system. I don't
write regularly enough to require a blog where one article is linked to the
next. Instead, I mostly update the main page of the site and manually link to
the different articles I write. The sitegenerator isn't particularly good at
this. Additionally, placing articles in their own Git repositories is
cumbersome because I have to work in two different directories and ensure the
article is committed and pushed. This feature is great for groups, but when
working alone, it's just additional steps without any real benefit.

So, I thought about what I really needed, and it turned out to be quite simple.
I wanted to write articles with simple formatting—like Markdown—and use
templates to turn them into acceptable-looking HTML. I needed an easy way to
add pictures and other elements, and that was about it.

I rewrote the sitegenerator into
[sitegenerator2](https://github.com/nbkr/sitegenerator2). It's now an even
simpler tool. It walks through the content directory, copies everything except
the `.md` files into the build directory, converts each `.md` file into an
`.html` file, combines it with the template, and also puts it into the build
directory. Easy, except for one thing.

I wanted to set the template used for each content file. This way, I can apply
a special gallery template if I have an article with many images. With the old
sitegenerator, every article had a `vars.yaml` file where I could define such
settings without mixing them with the content. Now, with all content in a
single folder, the `vars.yaml` is out of place, so I needed a different
solution.

During my brief experience with Jekyll, I learned about 'front matter'—a YAML
section at the beginning of each content file. That sounded like a good idea.
Separating the content and front matter block isn't difficult, but I hoped the
'markdown-to-html' library I was using already supported it. The documentation
suggested it did, but no matter how I tried, I couldn't make it work. Even the
simplest examples gave me error messages indicating that the methods I was
using didn't exist. I updated the library, thinking the version might be too
old, but nothing changed.

Before reverting to manually separating content and front matter, I decided to
try ChatGPT. I had used it for finding information, but not for any real
projects. This seemed like a good opportunity. So, I asked ChatGPT about it.

ChatGPT suggested the same solution: manually separate front matter and content
and parse each as necessary. I pointed out that the library should be able to
do it on its own, and the AI adjusted the code accordingly. However, the same
error message appeared. ChatGPT then mentioned that there might be different
versions of the library. Again, this matched my thoughts. I searched around and
found that the issue was not with versions, but with branches. the library had
been branched to a new project which wasn't immediatelly obvious.

I informed ChatGPT about this, and it changed the code to reflect the new
library. I ran it, and it separated front matter and content just fine. But the
content wouldn't render. ChatGPT's script used an instance variable that simply
didn't exist. I could tell this much from the new library's documentation, but
I couldn't get it to work. The documentation and my brain didn't cooperate
well. I started looking into the library's code to understand it better, until
it occurred to me that I could ask ChatGPT about it. So, I formulated the
problem I was encountering, and ChatGPT returned with a fully working example.
Simple as that.

The rest of the script involved copying files around and calling the template
engine, and the sitegenerator2 was ready for use.

## Conclusion

### About simplicity
If I look at the resulting [sitegenerator2](https://github.com/nbkr/sitegenerator) it
a very simple tool. It doesn't even have 200 lines of code. So it feels like there is not much
to show, but on the other hand it does exactly what I need it to do. And there is certain
elegance in that, which I really like and recently came to appreciate more and more.

### Regarding ChatGPT
ChatGPT is like the famous [rubber duck](https://en.wikipedia.org/wiki/Rubber_duck_debugging) 
learned to talk. It's not - yet - on the same level as a human. A human would have anticipated what
I was trying to do and also tested his/her or their scripts. With ChatGPT it felt like I had to
wrangle the answers from it. It would tell me what I needed to know, but only if asked correctly. 
That feels a bit wired. But then again, if it felt any more human, I'd have to look out for
a 'free the AI slaves'-petition to sign. Which would be even more wired. These are strange times.

