# wtfgo

This repo is used to build and host [golang.wtf](https://golang.wtf), a site inspired by [git.wtf](https://git.wtf) in order to make help answer questions for common issues go developers may run into.

## Building

This site is generated using [hugo](https://gohugo.io), which will be required if you wish to generate it on your own when creating a post.  Below are a list of some useful commands:
```shell
# generate static site which will be located under the public directory
hugo

# serve static site locally
hugo serve

# serve static site including draft posts
hugo serve -D

# serve static site including posts with a publish date in the future
hugo serve -F
```

## Contributing

All contributions are welcome, whether it's an article or creating an issue with a suggestion of a potential article.  If you do open a PR with an article I do ask for the following to be included (assuming it doesn't already exist):
* A yaml file under `data/authors` with your GitHub username being the name of the file
* A 400x400px headshot (or similar) included under `static/media/authors` that is also named after your GitHub username

By doing this, and setting the author to the same name, your information will be displayed underneath the post for author information.

## Licensing
This site and all its content, including code, is licensed under [CC-BY-SA-4.0 License](LICENSE)