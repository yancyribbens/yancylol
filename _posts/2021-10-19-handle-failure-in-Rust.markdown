---
layout: post
title:  "Failure Handling in Rust"
date:   2021-10-19 14:02:00 +0100
categories: rust
---

I've been working on WeeRust recently which is an IRC client developed using the Rust programming language.  A problem recently prevented WeeRust from forwarding the message to me over IMAP because of a temporary network hiccup.  Reviewing the code I found an oversight where the `send_email()` method returns an `Option<u32>` when instead it should return a `Result<(), AppErr>` because `send_email()` can fail.  I would like to handle that failure gracefully instead of panic which is what happened the other night crashing my client and disconnecting me from the IRC network.  I added the question mark operator to the `send_email` method <a href="https://github.com/yancyribbens/weerust/commit/7e8c66a0ec26691b900b0b2bc769ab8392841728#diff-e574a8b5cb145a5a0d3829cd5290a736d64938f8b1af48de1d1686e7e319c123R81"> here </a> (more details about the question mark <a href="https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html"> here </a>.  The question mark operator combined with the signature change will now match either `Ok(t)` or `Err(e)`, and if the match is an `Err`, then the function will immediately return `Err(e.into())`(more on how into() works later).  From here there's a few options; we can create a retry strategy, hold the message in a buffer until it can be viewed later, inform the sender the message can't be delivered or some combination.  Since this application is very much a WIP I decided to just inform the sender the message can't be forwarded for the time being.

The other change this required is adding a new option to the enum

{% highlight rust %}
SmtpError(lettre::transport::smtp::Error)
{% endhighlight %}

Next, we need to add a new trait <i>From&lt;T&gt;</i> to the enum.  This new From trait does a value-to-value conversion.  In our case, it converts:

{% highlight rust %}
SmtpError(lettre::transport::smtp::Error)
{% endhighlight %}

To:

{% highlight rust %}
AppErr::SmtpError(error)
{% endhighlight %}

Here is the trait definition looks:

{% highlight rust %}
impl From<lettre::transport::smtp::Error> for AppErr {
  fn from(error: lettre::transport::smtp::Error) -> Self {
    AppErr::SmtpError(error)
  }
}
{% endhighlight %}

The newly introduced question mark operator can return something that is `AppErr::..` even though the original error was a different type.  Wrapping an error in a common type while still maintaining the original context information which can be useful for debugging.  This is a common pattern for designing error propagation in Rust as described <a href="https://doc.rust-lang.org/std/convert/trait.From.html#examples"> here </a>.  The full commit is <a href="https://github.com/yancyribbens/weerust/commit/7e8c66a0ec26691b900b0b2bc769ab8392841728"> here </a>.
