---
layout: post
title:  "The into method and error handling in Rust"
date:   2021-11-23 14:02:00 +0100
categories: rust
---

On [10/19]({% post_url 2021-10-19-handle-failure-in-Rust %}) I wrote about recoverable errors with the question mark operator.  The question mark calls the `into()` method when an error is matched.

{% highlight rust %}
match File::create("foo.txt") {
    Ok(t)  => t.write_all(b"Hello world!"),
    Err(e) => return Err(e.into()),
}
{% endhighlight %}

In WeeRust, I leverage the question mark operator to map multiple error types into a unified type <a href="https://github.com/yancyribbens/WeeRust/blob/master/src/error.rs#L2">AppErr</a>.  The type `lettre::transport::smtp::Error` and `async_imap::error::Error` are different types from different crates, however, when `into()` is called on either of these types, the type is transformed into `AppErr` with a variant of either `ImapError` or `SmtpError`.  More generally, `Into<U>` for `T` where `U` is `AppErr` and `T` is either `lettre::transport::smtp::Error` or `async_imap::error::Error`.

The Into implementation is provided by the <a href="https://doc.rust-lang.org/std/convert/trait.From.html"> From </a> trait.  For example, in WeeRust, `lettre::transport::smtp::Error` implements the `From` trait (for the AppErr enum) as follows:

{% highlight rust %}
 impl From<lettre::transport::smtp::Error> for AppErr {
      fn from(error: lettre::transport::smtp::Error) -> Self {
          AppErr::SmtpError(error)
      }
  }
{% endhighlight %}

After implementing the above example, calling into() on a type lettre::transport::smtp::Error will result in a variant `AppErr::SmtpError(lettre::transport::smtp::Error)` of type `AppErr`. 

<a class="credit" href="https://m4rw3r.github.io/rust-questionmark-operator"> Example credit </a>
