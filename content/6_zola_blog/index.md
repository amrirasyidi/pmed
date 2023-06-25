+++
title = "I Made a Blog!"
date = 2023-06-25
authors = ["Amri Rasyidi"]
description = "I made a blog with the help of Zola."
[taxonomies]
series = ["learn-in-public"]
tags = ["how-to"]
+++

I like to overcomplicate my own problem. This time I overcomplicate how I want to make a blog.

I like to overcomplicate my own problem. This, I think, force me to learn (unnecessary) things.

While there are blogging platform (or content management system in general), like wordpress, substack, medium, I choose to create a blog “from scratch”.

Why? Because I like to have more freedom and flexibility on what I’m making.

The drawback? You will need to spare time making the writing platform rather than just focus on the writing/content. And many others actually.

In this post, I’m going to share how I learn to make a blog using Zola.

And really, if you have a suggestion or advice on how I can make this blog better, just let me know.

# Zola 101

You can go to the official page to find the official definition. I’ll use my own definition based on my limited understanding.

Zola is a static site generator.

The generator part means, I think, it “compiles” the “raw” files into a “built” web pages. One of the most obvious “compilation” process is, the scss files you have will be a css files later after you built the webpages. But I think, there is folders and files hierarchy that needs to be obeyed.

This is why I’m not really made it from scratch, zola is a helper to compile my blog.

# Installing Zola

First thing first, to make a blog with zola, you need, zola, of course. You can check the installation guide [here](https://www.getzola.org/documentation/getting-started/installation/). I use Windows, so I go the the [github release page](https://github.com/getzola/zola/releases), as suggested by the installation page, and download the pre-built binaries.

<figure>
    {{ image( img="6_1_zola_exe.png", alt="Zola binaries in github", style="border-radius: 0px; width: 100%;" ) }}
    <figcaption>Zola binaries in github</figcaption>
</figure>

I extracted the binaries to `d:\program files\zola`, then add it to my PATH, to make it available globally on my machine.

<figure>
    {{ image( img="6_2_env_path.png", alt="Path Settings in Windows", style="border-radius: 0px; width: 100%;" ) }}
    <figcaption>Path Settings in Windows</figcaption>
</figure>

To check if it is working or not, open command prompt, then type in `zola init myblog`.

<figure>
    {{ image( img="6_3_zola_init.png", alt="Zola init in CLI", style="border-radius: 0px; width: 100%;" ) }}
    <figcaption>Zola init in CLI</figcaption>
</figure>

# Zola init

Now to make a very basic static web (or a blog in this case), you can make a new folder where you want to put all your writings in it. From there, you can open your favorite IDE, I go with VSCode, and start building from there.

The `zola init myblog` is essentially our `hello world`, after you press enter to all the prompt, the folder that you made earlier will be filled with the basic folder structure that needed by zola to create your blog.

I’ll stop here for the zola init, you can check the complete tutorial on the official page.

# Using Theme

Now things are getting real. This is the part where I “reset” my whole blog the most, because, even until now, I don’t really know how to properly use a theme.

The official documentation say that the easiest way to install and use a theme is to clone the theme repo to our `theme` folder. Then, on the deployment tutorial, for github, the best way is to add submodule of the desired theme.

I went with the first option, clone the theme repo to my `theme` folder, **then** I remove the git file in my clone theme folder. I did it because I edited some of the file in the theme, I added some additional scss script here and there. To make the changes reflected in my repo, I have to make a pull request to the master(?) branch of the original repo, otherwise, the blog will reflect the “default” from the themes. Due to my lack of understanding and experience in this, I just go for the workaround.

One thing to put in mind though is that, every theme has their own expected folder tree, this is what you need to discover. The one I have do not expect a section for the main content, in this case the blog post, so it should be directly under the `content` folder.

# Deployment

Before I went to far, I’ll say right now that I use netlify to publish my blog, I tried github pages, but I got credentials issue, I don’t know what happened.

I have no problem at all while deploying to netlify, you only need the netlify account. Before deploying, make sure to add one more file named `netlify.toml` inside the root, the template of the file can be copied from the official documentation, just make sure you have the matching `zola_version`. I downloaded zola version 0.17, therefore I put it there. I also don’t use the base and publish variables, I don’t remember why do, so depending on the theme of you choice, it may differ. My final `netlify.toml` looks like this:

```toml
[build]
    command = "zola build"

[build.environment]
    ZOLA_VERSION = "0.17.2"

[context.deploy-preview]
    command = "zola build --base-url $DEPLOY_PRIME_URL"
```
- -
That’s it, the very first version of my blog!

In the next post, I’ll tell you how I fixed the sidebar link, adding series and tags, and also add a subscription form.

Till, next time.

So long!