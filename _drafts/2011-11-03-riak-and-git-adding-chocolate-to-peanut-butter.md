---
layout: post
title: Getting Started with Erlang and Rebar
---

# {{page.title}}

<span class="meta">August 20 2011</span>

In this post I'll walk you slowly through the process of setting up your first [Erlang]() application using [Rebar]().  Before we begin let's double check your environment.

I have tried to keep the setup very simple. This makes the environment easier to use, maintain, and debug.

For an editor, I use [TextMate](http://macromates.com) with a [TextMate Clojure bundle](https://github.com/mmcgrana/textmate-clojure). TextMate is an unusual choice among Clojure hackers but has worked well for me. As you will see, I've designed my setup so that the editor is independent of the rest of the environment. So if you prefer another editor things will work fine.

I don't run any Clojure code from within the editor: both the REPL and scripts are run from the command line using the following `clj` script:

{% highlight sh %}
USER_CLJ_DIR=~/Clojure
JAVA="java -Xmx"`freembytes`"m"

if [ -z "$1" ]; then
  rlwrap $JAVA -cp `cljcp` clojure.contrib.repl_ln
else
  $JAVA -cp `cljcp` clojure.main $USER_CLJ_DIR/run.clj $@
fi
{% endhighlight %}
    

Running `clj` with no arguments boots into a Clojure REPL. Running `clj <path>` invokes the Clojure script at the given path.

`clj` relies on two other scripts. The first is `freembytes`, which determines how much free RAM my machine has to offer the JVM:

{% highlight sh %}
FREEPAGES=`vm_stat | head -2 | tail -1 | awk '{print $3}'`
echo "(4096*$FREEPAGES)/1048576" | bc
{% endhighlight %}

The second is `cljcp`, which builds a Java classpath:

{% highlight sh %}
USER_CLJ_DIR=~/Clojure
CLJ_STACKTRACE_JAR=~/.m2/repository/clj-stacktrace/clj-stacktrace/0.1.0/clj-stacktrace-0.1.0.jar

CP=./:src/:src/clj/:src/main/clojure/
CP=$CP:test/:src/test/clojure/
CP=$CP:bench/:example/:scratch/:classes/

for file in lib/*.jar; do
  CP=$CP:$file
done

CP=$CP:$USER_CLJ_DIR
CP=$CP:$CLJ_STACKTRACE_JAR

echo $CP
{% endhighlight %}

When `clj` starts a REPL, it looks for a `user.clj` on the classpath. I have the following in `~/Clojure/user.clj`:

{% highlight clj %}
(use 'clojure.contrib.repl-utils 'clj-stacktrace.repl)

(defn quit []
  (System/exit 0))
{% endhighlight %}

This provides access to the [`clj-stacktrace`](https://github.com/mmcgrana/clj-stacktrace) REPL utilities and defines a handy `quit` function.

When the `clj` runs script files, it uses `~/Clojure/run.clj`, which is as follows:

{% highlight clj %}
(use 'clj-stacktrace.repl)
(import 'clojure.lang.Compiler)

(let [script-path (first *command-line-args*)
      script-args (rest  *command-line-args*)]
  (binding [*command-line-args*  script-args
            *warn-on-reflection* true]
    (try
      (Compiler/loadFile script-path)
      (catch Exception e
        (pst-on *err* true e)
        (System/exit 1)))))
{% endhighlight %}

This script print warnings when the Clojure compiler is using Java reflection and pretty-prints any exception stacktraces with `clj-stacktrace`.

Finally, I manage dependencies using [Leiningen](https://github.com/technomancy/leiningen). You can find good instructions for installing and useing Leiningen on the linked site. The Leiningen command `lein deps` pulls all dependencies required by a project into the local `lib` director, which is then picked up by the `cljcp` script and therefore available for use at the REPL and for running scripts.
