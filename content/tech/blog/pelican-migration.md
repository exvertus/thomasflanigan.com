Title: Migrating my site to Pelican
Summary: A Site Restructuring Adventure
Date: 2022-06-06
Tags: tech, how-to, python, pelican
Slug: pelican-migration

I recently came across a nice theme I liked in Hugo called [Dimension](https://github.com/your-identity/hugo-theme-dimension),
based on the responsive [html5up theme](https://html5up.net/dimension) of the same name.
The thing I liked most about the theme is how, at least at first glance, I could fix a problem with my old site.
The old layout I had placed far too much emphasis on my tech skills and barely mentioned my passion for art and music.
While I love the tech skills I have, and while I see coding/configuring 
(well it's all configuration-as-code these days, so I guess I can just say 'coding' now)
can be a creative outlet and mode of expression in some ways, 
I often feel much more connected and free spending time at the piano or with a pen and paper than I have coding,
which can often be a means to more practical ends.  

So I really liked the idea of what you see on my homepage now,
but I quickly ran into problems trying to apply it to my site with hugo.
I recently spent a few weeks working on my web-chops, so I had some fresh html/CSS/JS skills I was eager to apply.
And ironically Hugo, the static site building that got me off the ground quickly with a site was now getting in the way.
In order to add serious customization to hugo, I would have to learn the go templating syntax and potentially some golang too.
I thought that might stretch my learning-applying loop a bit too much, so I went with another option instead...

### Pelican to the rescue

I took a look at the current options for static site generation that Python has to offer and settled on [Pelican](https://blog.getpelican.com/).
With this I would only have the jinja2 templating syntax to grapple with, something I had much more familiarity with.
And of course if I required any further customization (and I quickly found out *I would*), 
Pelican provides "sky's the limit" customization via their plugin interface, 
so with 11 years of on and off Python experience,
I was feeling pretty good about undergoing a bigger switch and migrating over.

### The real work was the customization

Migrating the content didn't really require anything, since Hugo and Pelican both can read markdown.
Some of my shortcodes didn't carry over from hugo, but it was pretty easy to replace that with the
[jinja2content](https://github.com/pelican-plugins/jinja2content) plugin instead.

The first wrinkle was the theme itself.
Since Dimension is a single-page theme that assumes a fairly flat blog-style layout, 
it didn't have a good submenu option for listing article-style pages separately out of the box,
so I needed to add a #article-menu div for when I needed that.
I added supporting CSS that hooked into the existing stretching on page-load and
had to wrestle with some alignment and padding issues:

```
#article-menu {
		display: -moz-flex;
		display: -webkit-flex;
		display: -ms-flex;
		display: flex;
		-moz-flex-direction: column;
		-webkit-flex-direction: column;
		-ms-flex-direction: column;
etc...
```

The second big problem was with Pelican itself.
Without a custom plugin, Pelican will flatten all the folders you identify as articles and pages.
While there is only one big article list, 
Pelican uses things like categories and tags to generate extra pages in order
to navigate a large article list sensibly (if you let it).
This can work well for a blog centered around one specific subject, 
but in my case I wanted three sub-index pages: art, music and tech. 
I organized everything in my content folder before writing my plugin (see screenshot below).
I didn't want to have to reorganize how I wanted to think of my site,
just to fit Pelican's existing model.
So I realized I had to disable the existing article and page generators in Pelican for starters:

```
def disable_page_writing(generators):
    """
    Disable normal article and page generation.
    The html5up Dimension theme fits better as index pages.
    """
    def generate_output_override(self, _):
        if isinstance(self, ArticlesGenerator):
            log.debug('Skipping normal article generation...')
        if isinstance(self, PagesGenerator):
            log.debug('Skipping normal pages generation...')

    for generator in generators:
        if isinstance(generator, (ArticlesGenerator, PagesGenerator)):
            generator.generate_output = types.MethodType(generate_output_override, generator)

def register():
    signals.all_generators_finalized.connect(disable_page_writing)
```

So I want to stick with a content layout that made sense *to me*, 
so my content folder looks something like this
(the content file for this page is highlighted in green since I'm in updating as I create this post):

![Content Directory](/images/posts/pelican-migration/dirlayout.png)

I wanted Pelican to recognize if I had an index file in a directory.
If I do, it should make an index.html, reflecting the same depth in output that is sent to the server.
So /index.md should generate /index.html, and /tech/index.md should generate /tech/index.html, etc.

And because Dimension is a single index.html design, 
I wanted any subpages and articles at the same depth of a given index.md to be generated in the content of that single page.
Dimension uses a script to show/hide pages as the user clicks within that page.
So for right now I really only need four big index.html files: one at the root url in addition to the ones at `/art` `/music` and `/tech`.
So for any subfolders with their own index.html, 
I needed it to include any pages at the same depth as buttons,
and any article-folders at the same depth in the article-menu.

The result was a new generator that still leveraged of Pelican's
stock ArticleGenerator and PageGenerator for *reading* the article and page markdown content,
but stops short of *generating* output under separate pages.
Instead, any articles and pages at the same depth get dumped into its index.html (including the one you are reading now).

So not surprisingly my solution to this was a new generator called an IndexGenerator:

```
class IndexGenerator(Generator):
    def generate_context(self):
        """
        Find all index.{ext} files in content folder
        Add them to self.context['generated_content']
        """
    def generate_output(self, writer):
        """
        For each index page, generate index.html with 
        articles and pages at the same depth.
        """
def register():
    signals.get_generators.connect(get_generators)
```

I'm glossing over the code here because there are quite a few lines, 
but you can look at the full source on [github](https://github.com/exvertus/thomasflanigan).
The two important hooks in Pelican generators are the generate_context and generate_output methods.
Pelican will not generate any output until all Generator class objects have finished generate_context.
This is how I am able to highjack generate_context code from the Article and Page Generators
without their generate_output methods creating the standard (in my case extra) articles and page output files.

While this felt like a bit of an adventure, I much happier using Pelican. 
The site-builder fits my content folder, not the other way around, but I didn't have to reinvent any wheels to get there either.
Pelican's plugin interface allowed me to pretty seamlessly use the pieces I needed and override what I didn't need.
For example when you click out of this article you'll be brought back to my tech page,
with the main pages *for tech* under the top buttons and articles in a section below.
### Next steps

I'll need to clean up a bit of code and rethink the settings a bit before I do,
but I've got shipping this work as separate projects on my to-do list so other Pelican users can use this plugin and theme too.

Pelican makes it pretty easy to ship Plugins as pip packages
and you can load themes in a similar way with 'pelican-themes --install'.
I will need to look into Poetry package manager first, and also tweak a few minor responsiveness issues I caused when I tweaked the Dimension theme (you may have noticed a couple already). I'll have more details and a couple new tech posts when I get there.
I will announce those future posts when I get to them on my [linkedin](https://www.linkedin.com/in/thomas-flanigan/).