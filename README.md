# younjoo0614.github.io

Personal academic site that combines:

- **Look &amp; feel** of [andrewbanchich/forty-jekyll-theme](https://github.com/andrewbanchich/forty-jekyll-theme) (HTML5 UP "Forty"): dark navy/teal palette, full-width banners, hamburger menu.
- **Content architecture** of [academicpages/academicpages.github.io](https://github.com/academicpages/academicpages.github.io): Jekyll collections for `publications`, `talks`, `teaching`, `portfolio`, plus a structured CV page.

The home page (`/`) is a CV-first landing that mirrors academicpages but is rendered with forty's styling.

## Local development

```bash
bundle install
bundle exec jekyll serve
```

Then open `http://localhost:4000`.

## Editing your content

| What you want to change          | File / directory                       |
| -------------------------------- | -------------------------------------- |
| Site title, bio, social links    | `_config.yml`                          |
| Top navigation menu              | `_data/navigation.yml`                 |
| Home page sections               | `index.html`                           |
| Full CV (Education, Work, ...)   | `_pages/cv.md`                         |
| Add a publication                | new `.md` file in `_publications/`     |
| Add a talk                       | new `.md` file in `_talks/`            |
| Add a course / teaching entry    | new `.md` file in `_teaching/`         |
| Color palette / typography       | `_sass/base/_vars.scss`                |
| Profile / banner imagery         | `assets/images/`                       |

Each collection item uses front matter compatible with the academicpages template (`title`, `date`, `venue`, `excerpt`, `permalink`, ...). See the seeded sample files for reference.

## Credits

- Forty design: [HTML5 UP](https://html5up.net) (CCA 3.0)
- Jekyll port of Forty: [Andrew Banchich](https://github.com/andrewbanchich/forty-jekyll-theme)
- Academic content structure inspired by [Academic Pages](https://github.com/academicpages/academicpages.github.io)
