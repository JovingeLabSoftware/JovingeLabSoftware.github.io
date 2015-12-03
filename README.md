# Jovinge Lab Software Blog

Disruptions along a pathway paved with Big Data.

Presently using the slim jekyll theme. [Demo](http://syaning.com/slim).  I slightly edited main.css because default code font size was, IMHO, too big.

```
- code { padding: 1px 5px; font-family: "Courier New", Consolas, monopace, FangSong, STFangsong; }
+ code { padding: 1px 5px; font-size: 12px; font-family: "Courier New", Consolas, monopace, FangSong, STFangsong; }
```

Note that I did not need to install jekyll locally and build the site...I just cloned the slim github repository, copied everything into this repository, and then grabbed the 'compiled' style sheet from the demo site and added that to the `css` directory.  The main.css style sheet seems to be the only 'compiled' element in this template.  I also edited the site details is _config.yml, of course (and added a prose section for prose.io support--see details [here](http://jovingelabsoftware.github.io/blog/2015/12/03/getting-prose-io-to-work-with-our-github-user-blog/)).

### License

MIT
