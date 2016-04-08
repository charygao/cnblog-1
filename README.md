# CN Blog #

You need Hexo to build generate the blog. Right now it runs on Hexo 3.1.1.

`npm install`

When installed, go to blog folder and run `hexo server -i 127.0.0.1`. Enjoy your blog on `127.0.0.1:4000`.

# Drafts

If you push your post to branch with prefix `posts/` (e.g. `posts/graphite`), your post will be rendered and displayed at https://cnblogstage.firebaseapp.com. Of course if somebody else pushes to another `post/*` branch your stuff will be overwritten. Use it to verify that your post is OK, including attachments.
