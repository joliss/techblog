## Funding Gates' Tech Blog

## Setup

    rvm use 1.9.3
    rvm gemset create 'techblog'
    cd .

    # It should spit out: "using 1.9.3 with gemset techblog"

    bundle

### Style and Markup Guidelines

Following the [Rails API Documentation
Guidelines](http://guides.rubyonrails.org/api_documentation_guidelines.html),
where applicable, makes for good writing as well. Active voice, brevity, and
clarity are as important as content. If there is code inline make sure your
gist is of the correct language type so that syntax highlighting makes the most
of your contribution.

## Site Details

This is an Octopress site. Check out [Octopress.org](http://octopress.org/docs)
for guides and documentation.

Short-short version:

### Deploying

Site is deployed to Github pages. Simply run:

    rake deploy

If the site doesn't update correctly, run:

    rake gen_deploy

### Generation and such

    rake generate   # Generates posts and pages into the public directory
    rake watch      # Watches source/ and sass/ for changes and regenerates
    rake preview    # Watches, and mounts a webserver at http://localhost:4000
    rake gen_deploy # Generates and then deploys to github pages

### Blog Postings (probably 99% of the time)

    rake new_post["title"]
    # creates /source/_posts/title.markdown

### New Pages

    rake new_page[content/foo.markdown]
    # creates /source/content/foo.markdown

### Plugins

(add them here)

## License
(The MIT License)

Copyright © 2009-2011 Brandon Mathis

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the ‘Software’), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED ‘AS IS’, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.


#### If you want to be awesome.
- Proudly display the 'Powered by Octopress' credit in the footer.
- Add your site to the Wiki so we can watch the community grow.
