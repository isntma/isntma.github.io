<div align="center">

  # Chirpy Jekyll Theme

  A minimal, responsive, and powerful Jekyll theme for presenting professional writing.

  ![Gem](https://img.shields.io/gem/v/jekyll-theme-chirpy)
  [![Automatic build](https://github.com/isntma/isntma.github.io/actions/workflows/pages-deploy.yml/badge.svg?branch=main&event=push)](https://github.com/isntma/isntma.github.io/actions/workflows/pages-deploy.yml)
  [![GitHub](https://img.shields.io/github/license/isntma/isntma.github.io)](https://github.com/isntma/isntma.github.io/blob/main/LICENSE)
  [![Stars](https://img.shields.io/github/stars/isntma/isntma.github.io)](https://github.com/isntma/isntma.github.io/stargazers)
  
  [![Demo](https://raw.githubusercontent.com/cotes2020/chirpy-images/main/commons/devices-mockup.png)](https://cotes2020.github.io/chirpy-demo)

</div>

## Quick Start

Before starting, please follow the instructions in the [Jekyll Docs](https://jekyllrb.com/docs/installation/) to complete the installation of `Ruby`, `RubyGems`, `Jekyll`, and `Bundler`. In addition, [Git](https://git-scm.com/) is also required to be installed.

### Step 1. Creating a New Site

Create a new repository from the [**Chirpy Starter**](https://github.com/cotes2020/chirpy-starter/generate) and name it `<GH_USERNAME>.github.io`, where `GH_USERNAME` represents your GitHub username.

### Step 2. Installing Dependencies

Before running for the first time, go to the root directory of your site, and install dependencies as follows:

```console
$ bundle
```

### Step 3. Running Local Server

Run the following command in the root directory of the site:

```console
$ bundle exec jekyll s
```

Or run with Docker:

```console
$ docker run -it --rm \
    --volume="$PWD:/srv/jekyll" \
    -p 4000:4000 jekyll/jekyll \
    jekyll serve
```

After a while, navigate to the site at <http://localhost:4000>.

## Documentation

For more details on usage, please refer to the tutorial on the [demo website](https://cotes2020.github.io/chirpy-demo/).

## License

This work is published under [MIT](https://github.com/isntma/isntma.github.io/blob/main/LICENSE) License.
