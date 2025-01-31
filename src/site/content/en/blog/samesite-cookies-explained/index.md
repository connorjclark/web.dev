---
title: SameSite cookies explained
subhead:
  Secure your site by learning how to explicitly mark your cross-site cookies.
authors:
  - rowan_m
date: 2019-05-07
updated: 2019-06-13
hero: cookie-hero.jpg
description: |
  Learn how to mark your cookies for first-party and third-party usage the
  SameSite attribute. You can enhance your site's security by making use of
  SameSite's Lax and Strict values gaining some protection against CSRF attacks.
  Specifying the new None attribute allows you to explicitly mark your cookies
  for cross-site usage.
tags:
  - post
  - security
  - cookies
---

Cookies are one of the methods available for adding persistent state to web
sites. Each cookie is a `key=value` pair along with a number of attributes that
control when and where that cookie is used. You've probably already used these
attributes to set things like expiry dates or indicating the cookie should only
be sent over HTTPS. Servers set cookies by sending the aptly-named `Set-Cookie`
header in their response. For all the detail you can dive into
[RFC6265bis](https://tools.ietf.org/html/draft-ietf-httpbis-rfc6265bis-03#section-4.1),
but for now here's a quick refresher.

Say you have a blog where you want to display a "What's new" promo to your
users. Users can dismiss the promo and then they won't see it again for a while.
You can store that preference in a cookie, set it to expire in a month
(2,600,000 seconds), and only send it over HTTPS. That header would look like
this:

```text
Set-Cookie: promo_shown=1; Max-Age=2600000; Secure
```

<figure class="w-figure  w-figure--center">
  <img src="set-cookie-response-header.png" alt="Three cookies being sent to a
    browser from a server in a response" style="max-width: 60vw">
  <figcaption class="w-figcaption">
    Servers set cookies using the <tt>Set-Cookie</tt> header.
  </figcaption>
</figure>

When your reader views a page that meets those requirements, i.e. they're on a
secure connection and the cookie is less than a month old, then their browser
will send this header in its request:

```text
Cookie: promo_shown=1
```

<figure class="w-figure  w-figure--center">
  <img src="cookie-request-header.png" alt="Three cookies being sent from a
    browser to a server in a request" style="max-width: 60vw;">
  <figcaption class="w-figcaption">
    Your browser sends cookies back in the <tt>Cookie</tt> header.
  </figcaption>
</figure>

You can also add and read the cookies available to that site in JavaScript using
`document.cookie`. Making an assignment to `document.cookie` will create or
override a cookie with that key. For example, you can try the following in your
browser's JavaScript console:

```text
> document.cookie = "promo_shown=1; Max-Age=2600000; Secure"
< "promo_shown=1; Max-Age=2600000; Secure"
```

Reading `document.cookie` will output all the cookies accessible in the current
context, with each cookie separated by a semicolon:

```text
> document.cookie;
< "promo_shown=1; color_theme=peachpuff; sidebar_loc=left"
```

<figure class="w-figure  w-figure--center">
  <img src="document-cookie.png" alt="Javascript accessing cookies within the
    browser" style="max-width: 35vw;">
  <figcaption class="w-figcaption">
    JavaScript can access cookies using <tt>document.cookie</tt>.
  </figcaption>
</figure>

If you try this on a selection of popular sites you will notice that most of
them set significantly more than just three cookies. In most cases, those
cookies are sent on every single request to that domain, which has a number
of implications. Upload bandwidth is often more restricted than download for
your users, so that overhead on all outbound requests is adding a delay on your
time to first byte. Be conservative in the number and size of cookies you set.
Make use of the `Max-Age` attribute to help ensure that cookies don't hang
around longer than needed.

## What are first-party and third-party cookies?

If you go back to that same selection of sites you were looking at before, you
probably noticed that there were cookies present for a variety of domains, not
just the one you were currently visiting. Cookies that match the domain of the
current site, i.e. what's displayed in the browser's address bar, are referred
to as **first-party** cookies. Similarly, cookies from domains other than the
current site are referred to as **third-party** cookies. This isn't an absolute
label but is relative to the user's context; the same cookie can be either
first-party or third-party depending on which site the user is on at the time.

<figure class="w-figure  w-figure--center">
  <img src="cross-site-set-cookie-response-header.png" alt="Three cookies being
    sent to a browser from different requests on the same page"
    style="max-width: 60vw;">
  <figcaption class="w-figcaption">
    Cookies may come from a variety of different domains on one page.
  </figcaption>
</figure>

Continuing the example from above, let's say one of your blog posts has a
picture of a particularly amazing cat in it and it's hosted at
`/blog/img/amazing-cat.png`. Because it's such an amazing image, another person
uses it directly on their site. If a visitor has been to your blog and has the
`promo_shown` cookie, then when they view `amazing-cat.png` on the other
person's site that cookie **will be sent** in that request for the image. This
isn't particularly useful for anyone since `promo_shown` isn't used for anything
on this other person's site, it's just adding that overhead to the request.

If that's an unintended effect, why would you want to do this? It's this
mechanism that allows sites to maintain state when they are being used in a
third-party context. For example, if you embed a YouTube video on your site then
visitors will see a "Watch later" option in the player. If your visitor is
already signed in to YouTube, that session is being made available in the
embedded player by a third-party cookie—meaning that "Watch later" button will
just save the video in one go rather than prompting them to sign in or having to
navigate them away from your page and back over to YouTube.

<figure class="w-figure  w-figure--center">
  <img src="cross-site-cookie-request-header.png" alt="The same cookie being
    sent in three different contexts" style="max-width: 60vw;">
  <figcaption class="w-figcaption">
    A cookie in a third-party context is sent when visiting different pages.
  </figcaption>
</figure>

One of the cultural properties of the web is that it's tended to be open by
default. This is part of what has made it possible for so many people to create
their own content and apps there. However, this has also brought a number of
security and privacy concerns. Cross-site request forgery (CSRF) attacks rely on
the fact that cookies are attached to any request to a given origin, no matter
who initiates the request. For example, if you visit `evil.example` then it can
trigger requests to `your-blog.example`, and your browser will happily attach
the associated cookies. If your blog isn't careful with how it validates those
requests then `evil.example` could trigger actions like deleting posts or adding
their own content.

Users are also becoming more aware of how cookies can be used to track their
activity across multiple sites. However until now there hasn't been a way to
explicitly state your intent with the cookie. Your `promo_shown` cookie should
only be sent in a first-party context, whereas a session cookie for a widget
meant to be embedded on other sites is intentionally there for providing the
signed-in state in a third-party context.

## Explicitly state cookie usage with the `SameSite` attribute

The introduction of the `SameSite` attribute (defined in
[RFC6265bis](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-same-site-00))
allows you to declare if your cookie should be restricted to a first-party or
same-site context. It's helpful to understand exactly what 'site' means here.
The site is the combination of the domain suffix and the part of the domain just
before it. For example, the `www.web.dev` domain is part of the `web.dev` site.

{% Aside 'key-term' %} If the user is on `www.web.dev` and requests an image
from `static.web.dev` then that is a **same-site** request. {% endAside %}

The [public suffix list](https://publicsuffix.org/) defines this, so it's not
just top-level domains like `.com` but also includes services like `github.io`.
That enables `your-project.github.io` and `my-project.github.io` to count as
separate sites.

{% Aside 'key-term' %} If the user is on `your-project.github.io` and requests
an image from `my-project.github.io` that's a **cross-site** request.
{% endAside %}

Introducing the `SameSite` attribute on a cookie provides three different ways
to control this behaviour. You can choose to not specify the attribute, or you
can use `Strict` or `Lax` to limit the cookie to same-site requests.

If you set `SameSite=Strict` this means your cookie will only be sent in a
first-party context. In user terms, the cookie will only be sent if the site for
the cookie matches the site currently shown in the browser's URL bar. So, if the
`promo_shown` cookie is set as follows:

```text
Set-Cookie: promo_shown=1; SameSite=Strict
```

When the user is on your site, then the cookie will be sent with the request as
expected. However when following a link into your site, say from another site or
via an email from a friend, on that initial request the cookie will not be sent.
This is good where you have cookies relating to functionality that will always
be behind an initial navigation, such as changing a password or making a
purchase, but is too restrictive for `promo_shown`. If your reader follows the
link into the site, they want the cookie sent so their preference can be
applied.

That's where `SameSite=Lax` comes in by allowing the cookie to be sent with
these top-level navigations. Let's revisit the cat article example from above
where another site is referencing your content. They make use of your photo of
the cat directly and provide a link through to your original article.

```html
<p>Look at this amazing cat!</p>
<img src="https://blog.example/blog/img/amazing-cat.png" />
<p>Read the <a href="https://blog.example/blog/cat.html">article</a>.</p>
```

If the cookie has been set as so:

```text
Set-Cookie: promo_shown=1; SameSite=Lax
```

When the reader is on the other person's blog the cookie **will not be sent**
when the browser requests `amazing-cat.png`. However when the reader follows the
link through to `cat.html` on your blog, that request **will include** the
cookie. This makes `Lax` a good choice for cookies affecting the display of the
site with `Strict` being useful for cookies related to actions your user is
taking.

{% Aside 'caution' %} Neither `Strict` nor `Lax` are a silver bullet for your
site security. Cookies are sent as part of the user's request and you should
treat them the same as any other user input. That means sanitizing and
validating the input. Never use a cookie to store data you consider a
server-side secret. {% endAside %}

Finally there is the option of not specifying the value which has previously
been the way of implicitly stating that you want the cookie to be sent in all
contexts. In the latest draft of
[RFC6265bis](https://tools.ietf.org/html/draft-ietf-httpbis-rfc6265bis-03) this
is being made explicit by introducing a new value of `SameSite=None`. This means
you can use `None` to clearly communicate you intentionally want the cookie sent
in a third-party context.

<figure class="w-figure  w-figure--center">
  <img src="samesite-none-lax-strict.png" alt="Three cookies labelled None,
    Lax, or Strict depending on their context" style="max-width: 60vw;">
  <figcaption class="w-figcaption">
    Explicitly mark the context of a cookie as <tt>None</tt>, <tt>Lax</tt>, or <tt>Strict</tt>.
  </figcaption>
</figure>

{% Aside 'objective' %} If you provide a service that other sites consume such
as widgets, embedded content, affiliate programmes, advertising, or sign-in
across multiple sites then you should use `None` to ensure your intent is clear.
{% endAside %}

## Changes to the default behavior without SameSite

[Chrome 76](https://chromestatus.com/features/schedule) introduces a new
`same-site-by-default-cookies` flag. Setting this flag will shift the default
treatment of cookies to apply `SameSite=Lax` if no other `SameSite` value is
provided. This is a move towards providing a more secure default and one that
makes the intended purpose of cookies clearer to users. If you want to delve
into the details, check out Mike West's
["Incrementally Better Cookies"](https://tools.ietf.org/html/draft-west-cookie-incrementalism-00).

With this flag enabled in Chrome, both **new and existing cookies** without the
`SameSite` attribute will be restricted to the same site the user is browsing.
If you have cookies that need to be available in a third-party context, then you
must declare that to the browser and the user by marking them as
`SameSite=None`. You will want to apply this when setting new cookies and
actively refresh existing cookies even if they are not approaching their expiry
date.

{% Aside 'note' %} If you rely on any services that provide third-party content
on your site, you should also check with the provider that they are updating
their services. You may need to update your dependencies or snippets to ensure
that your site picks up the new behavior. {% endAside %}

These changes are backwards-compatible with browsers that have correctly
implemented earlier versions of the `SameSite` attribute, or just do not support
it at all. By applying these changes to your cookies, you are making their
intended use explicit rather than relying on the default behavior of the
browser. Likewise, any clients that do not recognize `SameSite=None` as of yet
should ignore it and carry on as if the attribute was not set.

Additionally in Chrome, if you also enable the
`cookies-without-same-site-must-be-secure` flag then you must also specify
`SameSite=None` cookies as `Secure` or they will be rejected. Note, this flag
won't have any effect unless you also have `same-site-by-default-cookies`
enabled.

{% Aside 'warning' %} At the time of writing, the network library on iOS and Mac
incorrectly handles unknown `SameSite` values and will **treat any unknown
value** (including `None`) as if it was `SameSite=Strict`, which affects Safari
on Mac and browsers wrapping WebKit on iOS (Safari, Chrome, Firefox, and
others). This should be fixed in an upcoming release and may be available in the
Tech Preview now. You can track their progress in the
[WebKit Bugzilla #198181](https://bugs.webkit.org/show_bug.cgi?id=198181).
{% endAside %}

### Behavior with `same-site-by-default-cookies` enabled

```text
Set-Cookie: promo_shown=1
```

{% Compare 'worse', 'No attribute set' %} If you send a cookie without any
`SameSite` attribute specified. {% endCompare %}

```text
Set-Cookie: promo_shown=1; SameSite=Lax
```

{% Compare 'better', 'Default behavior applied' %} Chrome will treat that cookie
as if `SameSite=Lax` was specified. {% endCompare %}

### Behavior with `cookies-without-same-site-must-be-secure` enabled

```text
Set-Cookie: widget_session=abc123; SameSite=None
```

{% Compare 'worse', 'Rejected' %} Setting a cookie without `Secure` **will be
rejected**. {% endCompare %}

```text
Set-Cookie: widget_session=abc123; SameSite=None; Secure
```

{% Compare 'better', 'Accepted' %} You must ensure that you pair `SameSite=None`
with the `Secure` attribute. {% endCompare %}

{% Aside 'objective' %} These flags are intended to encourage more secure
defaults, but you shouldn't rely on a browser's default behavior. Best practice
is to always be explicit in stating the attributes so your intent is clear.
{% endAside %}

## Designing for `SameSite` cookies

A core practice in software design is the
[Separation of Concerns](https://en.wikipedia.org/wiki/Separation_of_concerns)
which essentially states that each part of your system should have one clear
purpose. This is more simply put in the
[Unix philosophy](https://en.wikipedia.org/wiki/Unix_philosophy) which says,
"Make each program do one thing well". By adding the `SameSite` attribute to
your cookies you are creating the distinction between first-party and
third-party usage, but a third-party cookie can still be used in a first-party
context. It might seem simpler to just have the single cookie, but now you have
one component doing two jobs.

If you have one cookie that's providing functionality in both contexts, for
example perhaps providing a user identifier for your main site and for an
embeddable widget, then consider separating these into separate cookies. As
browsers and users start to change how they accept and manage cookies providing
this distinction makes your site more robust in scenarios where third-party
cookies are blocked or cleared.

## What should I do to enable `SameSite` today?

The majority of languages and libraries support the `SameSite` attribute for
cookies, however the addition of `SameSite=None` is still relatively new which
means that you may need to work around some of the standard behavior for now.
These are documented in the
[`SameSite` examples repo on GitHub](https://github.com/GoogleChromeLabs/samesite-examples).
If your particular use case is missing, please raise issue or submit a pull
request with your solution.

_Kind thanks for contributions and feedback from Lily Chen, Malte Ubl, Mike
West, Rob Dodson, Tom Steiner, and Vivek Sekhar_

_Cookie hero image by
[Pille-Riin Priske](https://unsplash.com/photos/UiP3uF5JRWM?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
on
[Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)_
