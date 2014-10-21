---
title: Email Repair
date: 2014-08-15
tags: rubygems
authors: Robert Fletcher
---

## Why

At ChalkSchools, we allow users to upload a group of contacts to send forms to.
In the past, we didn't do a very good job of validating those emails, so we
might end up with the likes of "foo" or "not-an-email" in our database. We also
found almost-emails like "actually-an-email@gmail" or "whatevs-yahoo.com".
While some people advocate for [not validating
emails](http://www.mdswanson.com/blog/2013/10/14/how-not-to-validate-email-addresses.html),
we want to better communicate to our users when their data is incomplete. We
intend to start displaying this to them on the front end, (perhaps with the
assistance of the [mailcheck library](https://github.com/mailcheck/mailcheck))
but we've also got a lot of existing records in our database that we'd like to
clean up. Hence, the [email repair
gem](https://github.com/ChalkSchools/email-repair).
READMORE
## What

Email Repair does its best to make sense of whatever string you pass to it, and
returns either a pretty good guess at an email or `nil` if it fails. Here are a
few examples:

```ruby
require 'email_repair'
mechanic = EmailRepair::Mechanic.new

mechanic.repair('blah@gmail.com') # => 'blah@gmail.com', of course

# add TLD for common domain names
mechanic.repair('bloo@yahoo') # => 'bloo@yahoo.com'

# strip out extra spacing in the email
mechanic.repair('ble e @fo o.com') # => 'blee@foo.com'

# fix letter swaps for common domains
mechanic.repair('blagh@htomail.com') # => 'blagh@hotmail.com'

# find an email in a string
mechanic.repair('Steve Holt <steve@holt.com>') # => 'steve@holt.com'
mechanic.repair('Mr. Email,a@email.com,b@email.com') # => 'a@email.com'
mechanic.repair("o'malley, o'malley@omalley.com") # => "o'malley@omalley.com"

mechanic.repair('not an email') # => nil
```

<br />
You can [check out our
specs](https://github.com/ChalkSchools/email-repair/blob/master/spec/lib/email_repair/mechanic_spec.rb)
for some more examples of what it can (and can't) handle.

## How

Email Repair runs a [series of
repairs](https://github.com/ChalkSchools/email-repair/blob/master/lib/email_repair/mechanic.rb)
on the string passed in, fixing common substitutions and typos, stripping out
spaces, downcasing, checking for common domain mishaps, and finally scanning
the result with an email regular expression to pull out anything in the result
that resembles an email. It's far from robust at the moment, but running it
across our own data, we managed to recover quite a few email addresses, as well
as clean house.
