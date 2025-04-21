---
layout: post
title:  "Welcome to Jekyll!"
date:   2024-05-12 14:36:03 +0700
categories: jekyll update
---
You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

Markdown cheatsheet is [here](https://gist.github.com/roachhd/779fa77e9b90fe945b0c).

Shell example
```bash
grpcurl grpc.galactica.crptmax.com:22443 list
curl -X GET "https://api.galactica.crptmax.com/cosmos/staking/v1beta1/validators?status=BOND_STATUS_BONDED"
```

Config example
```
mail {
	# See sample authentication script at:
	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript

	# auth_http localhost/auth.php;
	# pop3_capabilities "TOP" "USER";
	# imap_capabilities "IMAP4rev1" "UIDPLUS";

	server {
		listen     localhost:110;
		protocol   pop3;
		proxy      on;
	}

	server {
		listen     localhost:143;
		protocol   imap;
		proxy      on;
	}
}
```

Run `jekyll` locally:

```bash
bundle exec jekyll serve
```
