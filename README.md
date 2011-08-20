Google App Engine Mini Profiler for Java
====================================

 - [Features](#features)
 - [Demo](#demo)
 - [Getting Started](#getting-started)
   - [Dependencies](#dependencies)
   - [Installation](#installation)
   - [Configuration](#configuration)
 - [Instrumenting your code](#instrumenting-code)

About
-----

This is a "mini profiler" for the Google App Engine Java runtime.

It's heavily inspired by

 - [mvc-mini-profiler][] - .NET Mini Profiler that started it all.
 - [gae\_mini\_profiler][gaeminiprofiler] - Python Mini Profiler for the Google App Engine Python runtime.

[mvc-mini-profiler]: http://code.google.com/p/mvc-mini-profiler
[gaeminiprofiler]: https://github.com/kamens/gae_mini_profiler

It's released under the [MIT](http://en.wikipedia.org/wiki/MIT_License) license.

<a name="features"></a>
Features
--------

 - Live profiling of any request _in production_:
 
   - Provides a basic Java profiler used to explicitly profile sections of your code.   
   - Captures [Appstats][] data if the `AppstatsFilter` is running.
   
 - Capture of profiles for requests that redirected to the current request as well as any `XMLHttpRequests` that happen after the page loads _as they happen_.
   
 - Granular control over when profiling happens and who can see the results:
 
   - Limit to a subset of URLs,
   - Limit to app administrators,
   - Limit to certain app users (by email),
   - Any combination of the above.
   
 - Simple configuration:
   
   - A single `.jar` file (and a dependency on the [Jackson][] JSON library).
   - A new `<servlet>` and `<filter>` in your `web.xml` file.
   - An include in the `<head>` of your page template.
   
[Appstats]: http://code.google.com/appengine/docs/java/tools/appstats.html
[Jackson]: http://jackson.codehaus.org/

<a name="demo"></a>
Demo
----

A demo app (with profiling enabled for anyone) is at [http://gae-java-mini-profiler.appspot.com/]().

<img src="http://gae-java-mini-profiler.appspot.com/images/screenshot-1.png" alt="Example Screenshot" style="max-width: 480px;">

<a name="getting-started"></a>
Getting Started
---------------

<a name="dependencies"></a>
### Dependencies

 - `appengine-api-1.0-sdk` (you should already have this in your project)
 - `appengine-api-labs` (you should already have this in your project)
 - `servlet-api` (you should already have this in your project)
 - `jackson-core-asl`
 - `jackson-mapper-asl`

<a name="installation"></a>
### Installation

Clone the source from here and build it using [maven](http://maven.apache.org/).

Then copy the `gae-mini-profiler-1.0.0.jar` file (and the two Jackson jars) to your `WEB-INF/lib` folder.

<a name="configuration"></a>
### Configuration

Add the `ca.jimr.gae.profiler.MiniProfilerServlet` and `ca.jimr.gae.profiler.MiniProfilerFilter` to your `web.xml`.

    <servlet>
      <servlet-name>miniprofiler</servlet-name>
      <servlet-class>ca.jimr.gae.profiler.MiniProfilerServlet</servlet-class>
    </servlet>
    <servlet-mapping>
      <servlet-name>miniprofiler</servlet-name>
      <url-pattern>/gae_mini_profile/*</url-pattern>
    </servlet-mapping>
    <filter>
      <filter-name>miniprofiler-filter</filter-name>
      <filter-class>ca.jimr.gae.profiler.MiniProfilerFilter</filter-class>
    </filter>
    <filter-mapping>
      <filter-name>miniprofiler-filter</filter-name>
      <url-pattern>/*</url-pattern>
    </filter-mapping>
    
Note: The `MiniProfilerFilter` __MUST__ be before the `AppstatsFilter` if you want to have Appstats show up.

There are a bunch of `initParameter` params you can set to configure the profiler:

__Servlet Parameters__

<table border="1" cellpadding="5" cellspacing="0" width="100%">
 <thead>
  <tr><th width="25%">Parameter</th><th width="75%">Description</th></tr>
 </thead>
 <tbody>
  <tr><td><code>maxStackFrames</code></td><td>The maximum number of stack frames to show in the Appstats stack traces.  The default is to show all of them.</td></tr>
  <tr><td><code>htmlIdPrefix</code></td><td>Prefix to use for HTML ids generated by the profiler.  This <strong>MUST</strong> match the <code>htmlIdPrefix</code> in the filter definition. The default is <code>&quot;mp&quot;</code>.</td></tr>
  <tr><td><code>resourceCacheHours</code></td><td>Number of hours to cache the static resources generated by the profiler in the browser.  The default is not to cache at all (0 hours).</td></tr>  
 </tbody>
</table>

_Filter Parameters_

<table border="1" cellpadding="5" cellspacing="0" width="100%">
 <thead>
  <tr><th width="25%">Parameter</th><th width="75%">Description</th>
 </thead>
 <tbody>
  <tr><td><code>servletURL</code></td><td>The base URL that the servlet is mapped to.  This <strong>MUST</strong> match the URL in the <code>&ltservlet-mapping&gt;</code> specified for the <code>MiniProfilerServlet</code>.  The default is <code>/gae_mini_profile/</code>.</td></tr>
  <tr><td><code>restrictToAdmins</code></td><td>Whether to restrict profiling to app admins.  The default is false.</td></tr>
  <tr><td><code>restrictToEmails</code></td><td>Comma-delimited list of emails of app users to restrict profiling to.  The default is no restriction.</td></tr>
  <tr><td><code>restrictToURLs</code></td><td>Comma-delimited list of regular expressions of URL patterns that profiling should be done on.  This can be used to further limit the scope of the filter mapping specified in the <code>web.xml</code>. The default is no restriction.</td></tr>
  <tr><td><code>dataExpiry</code></td><td>How many seconds to keep profile data around in Memcache.  The default is 30 seconds.</td></tr>
  <tr><td><code>htmlIdPrefix</code></td><td>Prefix to use for HTML ids generated by the profiler.  This <strong>MUST</strong> match the <code>htmlIdPrefix</code> in the servlet definition. The default is <code>&quot;mp&quot;</code>.</td></tr>
 </tbody>
</table>

At the bottom of the `<head>` in your page (usually in whatever global template you are using), you must output
the contents of the `mini_profile_includes` request attribute.  This attribute will be `null` if the profiler
did not run for this request.  E.g.

    <head>
      <!-- Other Stuff would go here -->
      ${mini_profile_includes}
    </head> 
    
If you are already including jQuery and/org jQuery Templates on your page, this include needs to happen _after_.  If jQuery or jQuery Templates are not already included on the page, they will be.
    
### Start up your app!
    
And that's it.  When you run your application, depending on what restrictions you have set, you will see profiling stats showing
up in the left-hand corner of the page.

<a name="instrumenting-code"></a>
Instrumenting your code
-----------------------

Odds are you will want more than just the elapsed time for the entire request.

You can use the `MiniProfiler` class to record execution times for sections of your code.

The main method to call is `MiniProfiler.step( String stepName )`.  This will start a new profiling step, and return an object that
you can later call the `close()` method on to finish the step. These steps can be nested to create a tree-structure.

If the profiler is inactive, the steps won't do anything.

    Step step1 = MiniProfiler.step("Big things happening");
    try
    {
      Step step2 = MiniProfiler.step("Sub-Step 1");
      try
      {
        // Do some work
      }
      finally
      {
        step2.close();
      }
      Step step3 = MiniProfiler.step("Sub-Step 2");
      try
      {
        // Do some work
      }
      finally
      {
        step3.close();
      }
    }
    finally
    {
      step.close();
    }

This will show up in the profiler UI as something like (different numbers obviously):

    Name                        Duration (ms)   Self (ms)   Offset (ms)
    -------------------------------------------------------------------
    Request                     100.00          10.00       0 
      Big things happening      90.00           15.00       10.00
        Sub-Step 1              35.00           35.00       15.00
        Sub-Step 2              40.00           40.00       50.00        