# secur.ity-pro.be

This is my personal blog, using jekyll.

Theme is loosly based on the default minima theme. 

## site resources

Site resources are in a [different repo](https://github.com/wknd/site-resources), when that updates update this repo with:
```
git submodule update --remote
git commit -a -m 'resources update'
git push origin gh-pages
```
That repo contains all the ```_data _includes _layouts _sass``` directories. 
A problem is that ```_data``` references assets which are only in this github pages repo. Will this cause issues when I create other repositories for individual projects?

## images

I'm using the default imagemagick stuff in ubuntu and standard gnu find. Other than that there is no dependencies for this little image processing script. 
In case you don't have it already
```
sudo apt-get install image-magick
```
That should provide the ```mogrify``` binary.

When you add new images you should run the ``` minifyimages.sh``` script. The script will take all the images in the ```assets/images-original/``` folder, minify them and put them in the ```assets/images/``` folder if they don't already exist. It will also create the required subdirectories as needed.
This could possibly be further automated with some git hooks but that is a bit of a pain.

