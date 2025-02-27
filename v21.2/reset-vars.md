---
title: RESET (session variable)
summary: The SET statement resets a session variable to its default value.
toc: true
docs_area: reference.sql
---

The `RESET` [statement](sql-statements.html) resets a [session variable](set-vars.html) to its default value for the client session.


## Required privileges

No [privileges](authorization.html#assign-privileges) are required to reset a session setting.

## Synopsis

<div>{% remote_include https://raw.githubusercontent.com/cockroachdb/generated-diagrams/release-21.2/grammar_svg/reset_session.html %}</div>

## Parameters

 Parameter | Description
-----------|-------------
 `session_var` | The name of the [session variable](set-vars.html#supported-variables).

## Example

{{site.data.alerts.callout_success}}You can use <a href="set-vars.html#reset-a-variable-to-its-default-value"><code>SET .. TO DEFAULT</code></a> to reset a session variable as well.{{site.data.alerts.end}}

{% include copy-clipboard.html %}
~~~ sql
> SET extra_float_digits = -10;
~~~

{% include copy-clipboard.html %}
~~~ sql
> SHOW extra_float_digits;
~~~

~~~
 extra_float_digits
--------------------
 -10
(1 row)
~~~

{% include copy-clipboard.html %}
~~~ sql
> SELECT random();
~~~

~~~
 random
---------
 0.20286
(1 row)
~~~

{% include copy-clipboard.html %}
~~~ sql
> RESET extra_float_digits;
~~~

{% include copy-clipboard.html %}
~~~ sql
> SHOW extra_float_digits;
~~~

~~~
 extra_float_digits
--------------------
 0
(1 row)
~~~

{% include copy-clipboard.html %}
~~~ sql
> SELECT random();
~~~

~~~
      random
-------------------
 0.561354028296755
(1 row)
~~~

## See also

- [`SET` (session variable)](set-vars.html)
- [`SHOW` (session variables)](show-vars.html)
