Title: Github Pages, Travis CI, and Pelican
Date: 2020-08-04 22:14
Category: DevOps
Tags: pelican, travis-ci, github-pages
Slug: github-pages-and-pelican
Authors: Alex Kagno
Summary: How to use Travis CI to publish a Pelican site automatically to Github Pages

## Intro

Much of this project is based off of Humberto's blog post linked in the [references](#references) below. Basically, I wanted a cheap(free) way of hosting a simple website for anything I might want to host. In addition to this, I wanted it to be at least mildly personalized website that wasn't one of the 5 avilable Github Pages themes. I prefer Python over Ruby anyway, and would like to get more experience with using it for things other than scripting, so here we are!

For reference, I did this on MacOS Catalina with Homebrew installed.

## Github Setup

First, you'll want to create a Github Pages repo. For your personal site, there doesn't seem to be a way to serve off of any other branch than `master`. Start by creating a different default branch for the repo. I'm doing this prior to going into Github and setting it up there, and I used `pelican` for my default repo.

Install Pelican, Markdown, and ghp-import: `pip3 install pelican markdown ghp-import`

Repository setup:
```bash
cd; mkdir git; cd git
mkdir username.github.io
cd username.github.io
pelican-quickstart
```

Answer the prompts as you want, until you get a `tasks.py` question. Answer yes and then answer no to the subsequent prompts until you get a question about Github Pages. Answer yes to this one(obviously).

Now you've got a site(kinda)!

My next step was to set up a `requirements.txt`, which we'll need for the TravisCI set up.

```
pelican
markdown
ghp-import
```

After that, I wanted to get some information pages established. Specifically, I made a `resume.md` and `about.md` page. I put those pages in `content/pages/` which is where they will be parsed and built.

Any content you write will need to go into `content/`.

Okay, so now that you have some content, you'll want to build this manually. First, make sure you've commited your new content to Github repo:

```bash
git add .
git commit -m "initial commit"
git push -u origin pelican
```

Then, within the root of your Github site, run `ghp-import`. This will build your website and then force the build site into the `master` branch on your repo, this is why we're storing the source files in another repo.

Now, you should be able navigate to `username.github.io` and see your (unoriginal) personal site! For customization, I'll leave that up to you. I don't want to document my setup because it's likely to change and I don't think it'll matter if that part is documented in the long run. If anything, that'll be a separate blog.

## Travis CI Setup

Now that you have a site that works, let's focus on getting Travis CI set up and configured to build your website automatically whenever you push to your source branch. This means you can update your website from Github without needing access to a build machine! 

Log into [Travis CI](https://travis-ci.org) with your Github account and connect your repo to it. There's plenty of documentation surrounding this specifically, so I don't want to put it here where it can get out of date.

Before Travis CI can manage pushes to your repo, which is how `ghp-import` works, you'll need to create a new SSH key which Travis CI will integrate pretty easily into the repo.

First, install the Travis CI CLI tool, here it is for MacOS, I trust you can find it for your platform of choice:

```bash
brew install travis
```

Then, you'll want to create a new SSH key in the root of your `pelican` branch repo directory (no password):

```bash
ssh-keygen -f publish-key
```

Finally, you'll use `travis` to encrypt and store the key within Travis CI, allowing your build containers to push your built site:

```bash
travis encrypt-file publish-key
```

(also, don't forget to do some clean up!):

```bash
rm publish-key publish-key.pub
```

Now that that's set up, let's get your `.travis.yml` committed, please read the comments:

```yaml
sudo: false
branches:
  only:
  - pelican
language: python
before_install:
# change the line bellow with the output of the command: travis encrypt-file publish-key but keep the final part: -out ~/.ssh/publish-key -d
- openssl aes-256-cbc -K $encrypted_120135e51708_key -iv $encrypted_120135e51708_iv -in publish-key.enc -out ~/.ssh/publish-key -d
- chmod u=rw,og= ~/.ssh/publish-key
- echo "Host github.com" >> ~/.ssh/config
- echo "  IdentityFile ~/.ssh/publish-key" >> ~/.ssh/config
# replace username/username with your username
- git remote set-url origin git@github.com:username/username.github.io.git
- git fetch origin -f master:master
install:
- pip install --upgrade pip
- pip install -r requirements.txt
script:
- make github
after_script:
- git checkout master
- echo "$CNAME_DOMAIN" > CNAME
- git add CNAME
- git commit -m "added CNAME"
- git push origin master
```

Everything from before `after_script:` is from Humberto's post linked in the [references](#references) section. The `after_script` section is what I use to ensure my CNAME persists across `ghp-import` pushes because I use a custom domain like a cool kid. If you don't need it, scrap the whole section, otherwise set the `CNAME_DOMAIN` in your Travis CI environment variables (Settings > Environment Variables).

Throw the above YAML into that file and commit both this and the encrypted SSL file to your repo:

```bash
git add .
git commit -m "travis files"
git push origin
```

After that commit, your Travis CI pipeline should kick in and start building you site automatically!

## Conclusion

I hope you enjoyed reading through the set up of my website, and my first blog post. I expect most of what I post here will be pseudo-documentation for what I work on randomly in my personal time.

Thanks for stopping by!

## References

1. [Primary documentation](https://humberto.io/blog/publishing-at-github-pages-with-pelican-and-travis-ci/)
2. [Pelican documenation](https://docs.getpelican.com/en/stable/)