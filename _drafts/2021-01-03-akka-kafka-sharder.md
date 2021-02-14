---
title: "Using Kafka as a replacement for Akka Cluster native sharding"
excerpt: "When you need Akka "
header:
#teaser: "assets/images/markup-syntax-highlighting-teaser.jpg"
tags:
- akka
toc: true
---

# What is Akka?
It all started with a prophecy. Once upon a time... well, actually not so long ago. In 2007, Pat Helland published his now-infamous article promptly called Life beyond Distributed Transactions: an Apostate's Opinion. He argued that there is a fundamental obstacle in creating systems that want to scale nearly-infinitely, if access to the mutable state is locked by a distributed transaction. The integrity of changes, achieved this way, comes at a huge cost: such system just won't scale above certain threshold, no matter how much more hardware you add. The key to infinite scaling is to use distributed lock-free entities to hold the state, that could use all additional hardware in a most efficient way.

Akka appeared in 2009. At its core is an implementation of actor model, as known from Erlang, rewritten in Scala. Actors are stateful entities, communicating with each other asynchronously, by passing messages around. Each actor is guaranteed to process just one message a time, allowing for lock-free mutable state updates.

On top of actors keeping their state in memory, there is Akka Persistence, adding robust event sourcing, and Akka Cluster with Sharding to distribute persistent actor on available cluster nodes. Backed by scalable database such as Cassandra and scalable streaming such as Kafka, the resulting platform is exactly what Pat Helland envisioned in his article - a platform for nearly-infinite scalable system.

It took few years to mature into industrial quality software, and now Akka is being successfully used highly concurrent event processing systems across wide variety of industries: from gambling to banking, and from postal logistics to IoT - where each millisecond in latency matters, and data is extremely valuable. Scaling such a system to process 10x times the current load is solved by adding hardware, but, generally, without rewriting any code.

# Observability of Akka
Observability is the capability of the system to expose its internals in a meaningful way. Akka itself is free open source software at its core (Apache license), but to use the great Cinnamon telemetry library, commercial Lightbend subscription is required, which may not be affordable for every team.

As alternative, there is a metrics library called Kamon, using attached agent and bytecode instrumentation, to expose Akka metrics. Kamon is suitable for testing and moderate loads, but by nature of its instrumentation, it adds too much overhead that can't be acceptable in high-load production environments.

Teams working with Akka usually chose to implement their own custom metrics for Akka in production.

# Prometheus
Prometheus is free open source (Apache) time-series database that is widely used to keep process metrics. Prometheus collectors for JVM could be enhanced with any kind of metrics to collect, using very low-overhead concurrent JVM primitives.

# Akka Sensors
Akka Sensors is a new free open source (MIT) Scala library that instruments JVM and Akka explicitly, using native Prometheus collectors, with very low overhead, for high-load production use.

https://github.com/jacum/akka-sensors

While the library itself is new, the approaches and techniques applied are distilled from many years of production experience, implementing ad-hoc custom Akka/Prometheus metrics development, and from some OSS projects.

Key measurements performed on running actors, Akka dispatchers, Cassandra client. An optional 'runnable watcher', configurable per dispatcher, keeps an eye on runnables, reporting stack traces of those rogue ones, hanging too long on non-reactive activities: e.g. waiting for locks, or doing blocking I/O.

Collected metrics are indeed not as extensive as with Cinnamon, at the moment, most notably lacking the automatic instrumentation for cluster/sharding and remote traffic between cluster nodes.

It comes with set of pre-configured Grafana boards and example application (http4s API + Akka + Cassandra).



Syntax highlighting is a feature that displays source code, in different colors and fonts according to the category of terms. This feature facilitates writing in a structured language such as a programming language or a markup language as both structures and syntax errors are visually distinct. Highlighting does not affect the meaning of the text itself; it is intended only for human readers.[^1]

[^1]: <http://en.wikipedia.org/wiki/Syntax_highlighting>

### GFM Code Blocks

GitHub Flavored Markdown [fenced code blocks](https://help.github.com/articles/creating-and-highlighting-code-blocks/) are supported. To modify styling and highlight colors edit `/_sass/syntax.scss`.

```css
#container {
  float: left;
  margin: 0 -240px 0 0;
  width: 100%;
}
```

{% highlight scss %}
.highlight {
margin: 0;
padding: 1em;
font-family: $monospace;
font-size: $type-size-7;
line-height: 1.8;
}
{% endhighlight %}

```html
{% raw %}<nav class="pagination" role="navigation">
  {% if page.previous %}
    <a href="{{ site.url }}{{ page.previous.url }}" class="btn" title="{{ page.previous.title }}">Previous article</a>
  {% endif %}
  {% if page.next %}
    <a href="{{ site.url }}{{ page.next.url }}" class="btn" title="{{ page.next.title }}">Next article</a>
  {% endif %}
</nav><!-- /.pagination -->{% endraw %}
```

```ruby
module Jekyll
  class TagIndex < Page
    def initialize(site, base, dir, tag)
      @site = site
      @base = base
      @dir = dir
      @name = 'index.html'
      self.process(@name)
      self.read_yaml(File.join(base, '_layouts'), 'tag_index.html')
      self.data['tag'] = tag
      tag_title_prefix = site.config['tag_title_prefix'] || 'Tagged: '
      tag_title_suffix = site.config['tag_title_suffix'] || '&#8211;'
      self.data['title'] = "#{tag_title_prefix}#{tag}"
      self.data['description'] = "An archive of posts tagged #{tag}."
    end
  end
end
```

### Code Blocks in Lists

Indentation matters. Be sure the indent of the code block aligns with the first non-space character after the list item marker (e.g., `1.`). Usually this will mean indenting 3 spaces instead of 4.

1. Do step 1.
2. Now do this:

   ```ruby
   def print_hi(name)
     puts "Hi, #{name}"
   end
   print_hi('Tom')
   #=> prints 'Hi, Tom' to STDOUT.
   ```

3. Now you can do this.

### Jekyll Highlight Tag

An example of a code blocking using Jekyll's [`{% raw %}{% highlight %}{% endraw %}` tag](https://jekyllrb.com/docs/templates/#code-snippet-highlighting).

{% highlight javascript linenos %}
// 'gulp html' -- does nothing
// 'gulp html --prod' -- minifies and gzips HTML files for production
gulp.task('html', () => {
return gulp.src(paths.siteFolderName + paths.htmlPattern)
.pipe(when(argv.prod, htmlmin({
removeComments: true,
collapseWhitespace: true,
collapseBooleanAttributes: false,
removeAttributeQuotes: false,
removeRedundantAttributes: false,
minifyJS: true,
minifyCSS: true
})))
.pipe(when(argv.prod, size({title: 'optimized HTML'})))
.pipe(when(argv.prod, gulp.dest(paths.siteFolderName)))
.pipe(when(argv.prod, gzip({append: true})))
.pipe(when(argv.prod, size({
title: 'gzipped HTML',
gzip: true
})))
.pipe(when(argv.prod, gulp.dest(paths.siteFolderName)))
});
{% endhighlight %}

{% highlight wl linenos %}
Module[{},
Sqrt[2]
4
]
{% endhighlight %}

### GitHub Gist Embed

An example of a Gist embed below.

<script src="https://gist.github.com/mmistakes/77c68fbb07731a456805a7b473f47841.js"></script>