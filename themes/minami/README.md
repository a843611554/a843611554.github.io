# hexo-theme-Minami

[![build](https://github.com/nanqinlang/SVG/blob/master/build%20passing.svg)](https://github.com/NewBugger/hexo-theme-Minami)
[![author](https://github.com/nanqinlang/SVG/blob/master/author-nanqinlang-lightgrey.svg)](https://github.com/NewBugger/hexo-theme-Minami)
[![license](https://github.com/nanqinlang/SVG/blob/master/license-GPLv3-orange.svg)](https://github.com/NewBugger/hexo-theme-Minami)

> [hexo-theme-Minami](https://github.com/NewBugger/hexo-theme-Minami) is a theme for Hexo, based on [hexo-theme-vexo](https://github.com/yanm1ng/hexo-theme-vexo)

## Demo
the style could be seen via [preview](https://preview.sometimesnaive.org) and [my blog](https://sometimesnaive.org)  
![Demo](https://github.com/NewBugger/hexo-theme-Minami/raw/master/demo.png)  
![Demo2](https://github.com/NewBugger/hexo-theme-Minami/raw/master/demo-article.png)


## Features
### Style
- Banner Image
- Flexible Topbar
- Simple Layout
- No Webfont
- Article Catalog
- Back-to-top Button

### Addons
- Nprogress: loading progress bar
- Prismjs: code highlight
- Gitment: comment system
- Google-Analytics: statistics tools


## Instructure
the following is the folder instructure :
```
- layout
  -- archive.ejs
  -- index.ejs
  -- layout.ejs
  -- page.ejs
  -- _partial
     --- archive.ejs
     --- catalog.ejs
     --- footer.ejs
     --- head.ejs
     --- header.ejs
     --- nav.ejs
     --- pager.ejs
     --- top.ejs
- source
  -- css
     --- _config.styl
     --- style.styl
     --- _partial
         ---- archive.styl
         ---- article.styl
         ---- catalog.styl
         ---- footer.styl
         ---- header.styl
         ---- markdown.styl
         ---- nav.styl
         ---- pager.styl
  -- js
     --- function.min.js
     --- gitment.min.js
     --- jquery.min.js
     --- nprogress.min.js
     --- prism.min.js
  -- partial
     --- imgs
         ---- banner.jpg (added by user)
         ---- catalog.png
         ---- favicon.ico (added by user)
         ---- menu.png
         ---- top.png
     --- mincss
         ---- gitment.min.css
         ---- nprogress.min.css
         ---- prism.min.css
- _config.yml
```


## Config
```yaml
excerpt_link: Read_More

# menu on the page
menu:
 Home: /
 Archive: /archive/

# article catalog
catalog: true

# gitment
gitment: false
gitment_owner: 
gitment_repo: 
gitment_oauth_id: 
gitment_oauth_secret: 

# google analytics
# footer.ejs
googleanalytics: false
googleanalytics_id: 
googleanalytics_uri: https://www.google-analytics.com/analytics.js

##############################################################
# the following is the variable of dir
# write the dir you use

# website property
# footer.ejs
website_name: 南琴浪博客
website_uri: sometimesnaive.org

# favicon.ico
# head.ejs
favicon: /deps/partial/imgs/favicon.ico

# banner image
# /archive.ejs and /page.ejs
uri_banner: /deps/partial/imgs/banner.jpg

# style.css
# head.ejs
uri_mincss_style: /deps/css/style.css

# function.min.js
# footer.ejs
uri_minjs_function: /deps/js/function.min.js

# jquery.min.js
# head.ejs
#uri_minjs_jquery: //cdn.bootcss.com/jquery/3.2.1/jquery.min.js
uri_minjs_jquery: /deps/js/jquery.min.js

# nprogress css+js
# head.ejs
#uri_mincss_nprogress: //cdn.bootcss.com/nprogress/0.2.0/nprogress.min.css
uri_mincss_nprogress: /deps/partial/mincss/nprogress.min.css
#uri_minjs_nprogress: //cdn.bootcss.com/nprogress/0.2.0/nprogress.min.js
uri_minjs_nprogress: /deps/js/nprogress.min.js

# prism css+js
# head.ejs
uri_mincss_prism: /deps/partial/mincss/prism.min.css
uri_minjs_prism: /deps/js/prism.min.js

# gitment css+js
# head.ejs
uri_mincss_gitment: /deps/partial/mincss/gitment.min.css
uri_minjs_gitment: /deps/js/gitment.min.js
```


## TODO
- Catagory
- Tag
- Internal Search


## License
GPLv3, author @NewBugger 

Original License: MIT
