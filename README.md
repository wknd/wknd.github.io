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


