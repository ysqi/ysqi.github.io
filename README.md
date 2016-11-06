# Yu Shuang Qi's Website
[![Build Status](https://travis-ci.org/ysqi/ysqi.github.io.svg?branch=master)](https://travis-ci.org/ysqi/ysqi.github.io)

This is the source for https://yushuangqi.com. I use [Hugo](http://hugo.spf13.com) to
take the source here and create a static site which can be found at [Ysqi](https://yushuangqi.com)


# Work Flow

By travis service, Run Grun deploy task after got git push hook. 

`clean dir` -> `hugo create static website` -> `html minifiler html` -> `git push`  

# License

The following files and directories including their contents are Copyright Yu Shuang Qi, or
their respective copyright holders:

* content/
* static/css/
* static/img/

All other directories and files are MIT Licensed unless clearly
designated otherwise.