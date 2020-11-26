This is the source code for my github user site on
https://github.com/tenbaht/tenbaht.github.io

The site is rendered using hugo instead of jekyll.

## new post

	hugo new posts/title-of-the-post.md

edit the file in content/posts, add title image in static/images.

	hugo serve

After checking, commit the local changes into the blog repository:

	git add .
	git commit
	git push

Finally, deploy the rendered pages to github pages:

	./deploy.sh


Themes:
- hugo dusk: gut, dunkel
