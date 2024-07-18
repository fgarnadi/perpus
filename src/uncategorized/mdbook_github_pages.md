# Publishing mdBook as Github Pages
<sub>2024-07-18</sub>

Searching for GitBook alternative that you can deploy yourself on Github Pages?

Using mdBook might be one of the solution that you can use. It's quite simple and even though it's
created using Rust, no prior knowledge of Rust is required to start creating pages using mdBook.

## Getting Started
### Install Rust
Even no Rust knowledge required, but to actually start, you need to install the Rust toolchain first
to install the mdBook program. To install the Rust toolchain, please refer to the
[installation guide](https://www.rust-lang.org/tools/install).

### Install mdBook
Using `cargo` it's quite simple to install mdBook.
```bash
cargo install mdbook
```

After that you can check if mdBook is properly using
```bash
mdbook --version
```

### Initialize Repository
Create your book repository in your desired directory with
```bash
mdbook init <book-name>
```
it will create a new directory `<book-name>` in your current directory.

Run the following command in the created directory to serve the book in you local machine.
```bash
mdbook serve # default to serve in 127.0.0.1:3000
```

## Publishing to Github Pages
Please refer to [Github Pages Quickstart](https://docs.github.com/en/pages/quickstart) to setup
the repository for Github Pages.

To simplify the process of publishing, there are github-actions that can be used :
- [peaceiris/actions-mdbook](https://github.com/peaceiris/actions-mdbook)
- [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)

You can read the repository for more details, but the excerpt is to create github-action workflow
that look like this in your repository (i.e in `.github/workflows/gh-pages.yml`).
This was taken from the repository examples and modified accordingly.

```yaml
# .github/workflows/gh-pages.yml
name: GitHub Pages

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-22.04
    permissions:
      contents: write
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - uses: actions/checkout@v4

      - name: Setup mdBook
        uses: peaceiris/actions-mdbook@v2
        with:
          mdbook-version: 'latest'

      - run: mdbook build

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v4
        if: github.ref == 'refs/heads/master'
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./book
```

As this workflow stated, this action will trigger when you push to `master` branch and will publish
the static file generated from the build to `gh-pages` branch that will be served as Github Pages.

## (Optional) Setting up Google Analytics
mdBook support additional custom tags that will be added to the build files.
To add this custom tags, in the root of your project create folder named `theme`, then create
a `handlebar` file named `head.hbs`. This file can be used to inject the Google Analytics script.

```html
<!-- theme/head.hbs -->

<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID"></script>
<script>
    window.dataLayer = window.dataLayer || [];
    function gtag() { dataLayer.push(arguments); }
    gtag('js', new Date());

    gtag('config', 'GA_MEASUREMENT_ID', {
        'anonymize_ip': true,
        'cookie_domain': window.location.hostname,
        'cookie_flags': "SameSite=None;Secure",
    });
</script>
```

Extra parameter in the config is to mitigate warning caused by Google Analytics when the page is
opened using Mozilla Firefox (taken from [Stack Overflow](https://stackoverflow.com/a/62569420)).

## References
- [mdBook Documentation](https://rust-lang.github.io/mdBook/)
- [Github Pages Quickstart](https://docs.github.com/en/pages/quickstart)
- [peaceiris/actions-mdbook](https://github.com/peaceiris/actions-mdbook)
- [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)
- [Stack Overflow answer for Google Analytics warning](https://stackoverflow.com/a/62569420)