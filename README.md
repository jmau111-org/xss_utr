# XSS Under The Radar

## Disclaimer

You won't find any `alert` here. Open the console.

## What is XSS? (in very simple words)

It's the act of injecting malicious scripts in a seemingly trusted website to send them to unsuspecting users.

There's a large range of XSS types and attacks. Here we'll only see those I consider a bit harder to understand/detect.

Even if devs care about security, as you would expect, attackers may still find ways to inject crap.

## Why attackers exploit XSS flaws

Here are some goals that can be achieved by exploiting XSS vulnerabilities:

* steal cookies & credentials
* redirect users to a phishing website
* disclose confidential information
* alter the publications
* spread malware
* install cryptominers or keyloggers
* harm the targeted website's reputation

The bad news is that XSS can be **stored**, so simply browsing a vulnerable website without interacting with it can still be dangerous.

## Misknown XSS

### IMG

`<img>` tags are often neglected when searching for XSS flaws.

```HTML
 <!DOCTYPE html>
<html lang="en">
 <head>
  <meta charset="UTF-8">
  <title>Demo malicious IMG event</title>
  <meta name="viewport" content="width=device-width,initial-scale=1">
 </head>
 <body>
  <h1>Demo IMG event</h1>
  <p>Open the source code</p>
  <!-- open the console -->
  <!-- any "on" event would work -->
  <img src=x alt="" class="blank" onerror=console.log("pawned!")>
 </body>
</html> 
```

More evasions are available [here](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html)

### SVG

Scalable Vector Graphics (SVG files) are XML-based documents that can execute JavaScript.

```HTML
<!DOCTYPE html>
<html lang="en">
 <head>
  <meta charset="UTF-8">
  <title>Demo malicious SVG</title>
  <meta name="viewport" content="width=device-width,initial-scale=1">
 </head>
 <body>
  <h1>Demo svg</h1>
  <p>Open the source code</p>
  <!-- open the console -->
  <embed src="data:image/svg+xml;base64, PD94bWwgdmVyc2lvbj0iMS4wIiA/Pgo8c3ZnIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+CiAgPHBhdGg+PC9wYXRoPgogIDxzY3JpcHQ+PCFbQ0RBVEFbY29uc29sZS5sb2coInB3bmVkISIpXV0+PC9zY3JpcHQ+Cjwvc3ZnPgo=">
 </body>
</html>
```

### PHP

While it's a classic one, it remains quite misknown by beginners:

```PHP
<!DOCTYPE html>
<html lang="en">
 <head>
  <meta charset="UTF-8">
  <title>Demo PHP XSS</title>
  <meta name="viewport" content="width=device-width,initial-scale=1">
 </head>
 <body>
  <p>
    <?php echo 'Does the URL ' . urldecode($_SERVER["REQUEST_URI"]) . ' even exist?!'; ?>
  </p>
 </body>
</html>
```


## False positives & useless patches

Static analysis is necessary but not sufficient. I've seen lot of security audits "revealing" XSS flaws pretty much everywhere, but not all could be demonstrated/exploited.

For example, I remember some audits on WordPress installations (old themes, not modern themes built with Gutenberg) spotting hundreds XSS in the theme files.

I've seen dev teams wrapping **all** WordPress utilities and other dynamic elements with "escapers" and "sanitizers" to fix the problem.

Be careful with such approach, as you will likely clutter the code, making it way less readable and maintainable, while the security benefits are questionable.

It's easy to miss many occurrences, and, as we saw in the demos, some DOM XSS are more difficult to spot than others.

Besides, this one does not fix the problem:

```php
<input type=text name=lol value=<?php echo esc_attr( $lol ); ?>>
```

## How to do it right, then?

### Some recommendations are deprecated

This security header might be counterproductive:

```
X-XSS-Protection: 1; mode=block
```

The purpose of this header is to re-enable the XSS-filter if it's disabled by the user. It's now deprecated, and you'd better use a valid [content security policy](https://developer.mozilla.org/en-US/docs/Web/Security/CSP/Introducing_Content_Security_Policy) (CSP) instead.

The only edge case where it might still make sense is for very old browsers (legacy).

### Efficient mitigations

#### Encode data on output

Depending on the context (HTML, JavaScript), you won't apply the same encoding, but some values are particularly sensitive, for example, `<` and `>`, and must be encoded before display. Otherwise, you app will be prone to attacks.

#### Validate input on arrival

In addition to encoding, input validation is critical. The recommended approach is to **ban/block invalid inputs** and raise an error. It might seem tedious, but many apps try to "make wrong inputs valid" or guess the expected value, which leads to all kinds of abuses.

App creators want to improve the experience and ensure users won't get frustrated by "impossible" forms to fill when they try to register. Thanks to modern languages, it's possible to raise errors without reloading the page every time, and you can provide more details about the expected formats to your users.

IMHO, uX is more about helping the users in their experience than producing oversimplistic and unsecure processes. If I can create accounts that easily bypass your policy or send unexpected values to store XSS payloads, it's not a good experience for me, especially if you ask me confidential documents once I'm registered.

#### How am I supposed to implement that as a developer?

In doubt, use robust and trusted libraries that are actively maintained. Manual whitelists and blacklists are prone to evasion in my experience.

#### Use a valid CSP

Here's a basic example by PortSwigger:

```
default-src 'self'; script-src 'self'; object-src 'none'; frame-src 'none'; base-uri 'none';
```

[Source: portswigger](https://portswigger.net/web-security/cross-site-scripting/preventing)

However, you will likely have to customize it according to your context. Use testing environments that replicate the real conditions to prevent unwanted display errors, as a strict policy may prevent some resources from loading.

### Find good payloads

* [XSS-Payload-without-Anything](https://github.com/hahwul/XSS-Payload-without-Anything)
* [XSS contexts](https://portswigger.net/web-security/cross-site-scripting/contexts)
* [DOM XSS wiki](https://github.com/wisec/domxsswiki/wiki)

### You will not find them all alone

As we saw, there are rare "Pokémons" that are more difficult to catch, so I would also recommend using automatic tools, as a pragmatic approach, for example, [XSS Hunter](https://xsshunter.com/) (there are many alternative scanners).

It's easy to miss lots of XSS when you do your tests manually. For example, blind XSS detection requires advanced configuration.
