## Massive Guide to Email Aliasing Tricks

By Taylor Christian Newsome

Email aliasing allows one mailbox to receive messages sent to many different variations of an address. These techniques are widely used for organization, testing, and development workflows. Many modern email providers support forms of aliasing or address normalization. Below is a comprehensive explanation of the most common and lesser known methods.

---

## Plus Addressing

Plus addressing is one of the most widely supported aliasing features. It works by allowing extra text to be added to the username portion of an email address using a plus symbol.

Example base address

```
example@example.com
```

Possible variations

```
example+1@example.com
example+github@example.com
example+newsletter@example.com
example+randomtext@example.com
```

All of these variations still deliver mail to the same mailbox associated with

```
example@example.com
```

Everything placed between the plus sign and the domain is treated as a tag. Many mail systems ignore that tag during delivery.

This method is commonly used for

* identifying which service sent an email
* organizing incoming messages
* automatically filtering messages into folders
* testing signup systems during development

For example, a developer might register on several platforms using different tagged addresses to see which service sends notifications or leaks the address later.

---

## Dot Aliasing

Dot aliasing is supported by certain email providers, especially Gmail. In these systems the periods inside the username are ignored during routing.

Base address

```
example@gmail.com
```

Valid variations

```
e.xample@gmail.com
ex.ample@gmail.com
exa.mple@gmail.com
exam.ple@gmail.com
examp.le@gmail.com
```

Even though they look different, they all map to the same mailbox because the mail system removes the dots internally before delivering the message.

The system interprets them as

```
example@gmail.com
```

This means the dots are purely cosmetic and can appear in many different combinations while still reaching the same inbox.

---

## Combining Dots With Plus Tags

Dot aliasing and plus addressing can be combined together to generate a very large number of address variations.

Example combinations

```
example+test@gmail.com
example+github@gmail.com
e.xample+alpha@gmail.com
ex.ample+beta@gmail.com
exa.mple+gamma@gmail.com
```

All of these variations still route to

```
example@gmail.com
```

The mail provider processes the address in two steps

1 remove the dots
2 ignore everything after the plus tag

This results in the base username being used for delivery.

---

## Gmail and Googlemail Domain Equivalence

Another lesser known variation exists with Gmail accounts.

Two domains are internally treated as identical

```
gmail.com
googlemail.com
```

A mailbox registered as

```
example@gmail.com
```

can also receive mail sent to

```
example@googlemail.com
```

When combined with other aliasing methods the variations multiply.

Examples

```
example@googlemail.com
example+test@googlemail.com
e.xample@googlemail.com
ex.ample+alpha@googlemail.com
```

All still arrive in the same Gmail inbox.

---

## Dot Position Variations

Another technique is changing the location of dots throughout the username.

Example

```
example@gmail.com
```

Possible placements

```
e.xample@gmail.com
ex.ample@gmail.com
exa.mple@gmail.com
exam.ple@gmail.com
examp.le@gmail.com
```

A username with six characters can produce dozens of possible dot combinations.

These are all normalized to the same internal username.

---

## Tagging Systems for Organization

Aliases can also be used to create structured tagging systems.

Example tagging patterns

```
example+shopping@gmail.com
example+banking@gmail.com
example+music@gmail.com
example+gaming@gmail.com
example+work@gmail.com
```

This allows automatic filtering rules such as

* messages containing +shopping moved to a shopping folder
* messages containing +work moved to a work folder

Many people use this method to track where their address was used.

---

## Subdomain Address Routing

Some email systems support routing addresses based on subdomains.

Example base mailbox

```
example@domain.com
```

Possible routed addresses

```
example@mail.domain.com
example@alerts.domain.com
example@support.domain.com
```

When configured correctly on the mail server all of these can forward to the same inbox.

This is more common in self hosted email environments.

---

## Catch All Email Addresses

A catch all mailbox is configured at the domain level and receives mail sent to any username at that domain.

Example domain

```
domain.com
```

Possible incoming addresses

```
anything@domain.com
random123@domain.com
serviceaccount@domain.com
testaccount@domain.com
```

All of these messages can be routed to a single mailbox such as

```
admin@domain.com
```

This method effectively creates unlimited email aliases.

It is widely used by developers, security researchers, and businesses managing many service accounts.

---

## Username Variants Through Case Insensitivity

Most email systems treat uppercase and lowercase letters as identical.

Example

```
example@gmail.com
Example@gmail.com
EXAMPLE@gmail.com
eXaMpLe@gmail.com
```

All of these are interpreted as the same mailbox.

Although it creates visual variations, it typically does not bypass account uniqueness checks because many platforms normalize case before storing addresses.

---

## Address Normalization in Modern Systems

Many websites now normalize email addresses before storing them. This process often includes

* removing dot characters
* removing plus tags
* converting domains such as googlemail to gmail
* converting letters to lowercase

This prevents multiple accounts from being created through simple alias tricks.

However the aliasing methods remain extremely useful for

* email organization
* spam tracking
* development testing
* debugging signup flows
* identifying data leaks

---

## Simple Summary

Email aliasing techniques include

Plus addressing
Adding text after a plus sign to create tagged variations

Dot aliasing
Placing periods inside the username where the mail system ignores them

Domain equivalents
Using alternate domains that map to the same mailbox

Catch all domains
Routing any address at a domain to one inbox

Case variations
Changing capitalization without affecting delivery

When combined, these techniques can create a very large number of address variations that all route to a single mailbox while still being easy to filter and organize. 📧
