---
layout: post
title:  "PHP: send datetime to JavaScript"
date:   2018-10-04 10:00:00 -0300
categories: [development , javascript]
tags: [php , javascript , datetime , iso8601 , api , rest , soap , json]
author: Gabriel Heming
---
Often when working with Rest API, or ajax requests, the `datetime` information must be sent from back-end to front-end.
Sometimes, to convert from received `datetime` format to JavaScript [`Date`][date-object] object supported format
might be really annoying.

In this way, questions like [this][stackoverflow-question-1] and [this][stackoverflow-question-2] can be found in Stack
Overflow.

Furthermore, the [MDN Date's manual][mdn-date-manual] has explained what happens with month format.

> **Note**: The argument monthIndex is 0-based. This means that January = 0 and December = 11.

PHP's `DateTime`, on the other hand, is 1-based. It means that JavaScript supports different formats of date and
sometimes the difference on JavaScript or even on PHP has to be handled. Moreover, that is probably why
[weird implementations][stackoverflow-question-3] to cope with that can be found.

Although date parts (eg.: _year_, _month_, _day_, etc...) can be sent in separated fields, it does not follow a good API
design. Even more, the main point here is to find a common format that is fully supported in both sides. Further, it is
necessary to understand which formats are supported by JavaScript `Date` object and that information can be found in
[MDN Date's manual][mdn-date-manual]

Therefore the constructor has a few ways to instantiate a `Date` object, only one constructor is going to be focused,
the same one as used in [`Date.Parse()`][mdn-date-parse].

> **dateString**: String value representing a date. The string should be in a format recognized by the Date.parse()
method (IETF-compliant RFC 2822 timestamps and also a version of ISO8601).

There is a note below `Date.Parse()` method that says:

> It is not recommended to use Date.parse as until ES5, parsing of strings was entirely implementation dependent. There
are still many differences in how different hosts parse date strings, therefore date strings should be manually parsed

Actually, ECMAScript 5 was released in December 2009, almost a decade ago. In this way, JavaScript `Date` can be created
using the well-known `ISO8601` format:

```
YYYY-MM-DDThh:mm:ssTZD
```

Each part of the format correspond to:

- YYYY = four-digit year;
- MM = two-digit month (01=January, etc.);
- DD = two-digit day of month (01 through 31);
- hh = two digits of hour (00 through 23) (am/pm NOT allowed);
- mm = two digits of minute (00 through 59);
- ss = two digits of second (00 through 59);
- TZD = time zone designator (Z or +hh:mm or -hh:mm).

The code below creates a `Date` object with the format above:

{% highlight javascript %}
var datetime = new Date("2018-10-03T12:00:00+00:00");
{% endhighlight %}

Console result:

```
Wed Oct 03 2018 09:00:00 GMT-0300 (Brasilia Standard Time)
```

The `ISO8601` format is a good choice because the timezone is carried with the datetime and it is not necessary to worry
about the timezone at all.

Actually, it is evident in the example above. The date is 12 pm on October 3, 2018, and the time was converted to 09 am
of GTM-0300 because the computer, where the code runs, was configured to _Brasilia Standard Time_.

But how could the proper datetime format be retrieved from PHP? Must that format be created? The answer is a simple no.
That is not necessary because PHP implements an "_almost powerful_" library called [`Date\Time`][datetime-book].

`Date\Time` has a bunch of classes, however, [`DateTime`][datetime-class], and its `interface`
[`DateTimeInterface`][datetime-interface], are going to be used here to reach the goals.

`DateTimeInterface` has a few constants to be used with the method `DateTime::format()` and it has the format
`ISO8601`. Yet it is named as `ISO8601`, the format, though, does not correspond to `ISO8601` and it is maintained "as
is" just for backward compatibility reasons.

> **DateTimeInterface::ISO8601**:
>
**Note**: This format is not compatible with ISO-8601, but is left this way for backward compatibility reasons. Use
DateTime::ATOM or DATE_ATOM for compatibility with ISO-8601 instead.

On the other hand, `DateTimeInterface` has three equivalent formats for `ISO8601`:
 - `ATOM`;
 - `RFC3339`;
 - `W3C`.

Those constants have the same format (`Y-m-d\TH:i:sP`) and can be used in the same way. Besides that, for internet
purpose, the constant `W3C` is going to be assumed.

Now, here is the code:

{% highlight php %}
//Timezone +0200
$date = new \DateTime('2018-10-03 10:00');
echo $date->format(DateTime::W3C);

//Timezone -0300
$date->setTimezone(new DateTimeZone('America/Sao_Paulo'));
echo $date->format(DateTime::W3C);
{% endhighlight %}

Output:

```
2018-10-03T10:00:00+02:00
2018-10-03T05:00:00-03:00
```

Now, we only have to integrate both codes, like a `JSON`/Rest API:

{% highlight php %}
$date = new \DateTime('2018-10-03 10:00');
$ret['datetime'] = $date->format(DateTime::W3C);
echo json_encode($ret['datetime']);
{% endhighlight %}

Even if the PHP is mixed with JavaScript:

{% highlight php %}
$date = new \DateTime('2018-10-03 10:00'); ?>
var datetime = new Date('<?= $date->format(DateTime::W3C); ?>');
{% endhighlight %}

## Microseconds

JavaScript `Date` and PHP `DateTime` support microseconds and the constant `RFC3339_EXTENDED` can be used as the proper
format.

{% highlight php %}
//now
$date = new \DateTime();
echo $date->format(DateTime::RFC3339_EXTENDED);
{% endhighlight %}

However, `RFC3339_EXTENDED` was only implemented in `PHP 7` as a `DateTimeInterface` `constant`. In this way, the
format has to be written as described below:

{% highlight php %}
//now
$date = new \DateTime();
echo $date->format('Y-m-d\TH:i:s.vP');
{% endhighlight %}

## Considerations

Meanwhile, this text shows a format to normalize datetime values between two specifics programming languages, 
PHP and JavaScript, it can be used as a guideline for any API that has to transport those values.

Additionally, API design guidelines are such powerful rules/techniques to provide sustained communication between two 
points. Well, that's another blog post for another time.

[date-object]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date
[stackoverflow-question-1]: https://stackoverflow.com/q/12254333/1628790
[stackoverflow-question-2]: https://stackoverflow.com/q/9755911/1628790
[stackoverflow-question-3]: https://stackoverflow.com/q/40916487/1628790
[mdn-date-manual]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date#Parameters
[mdn-date-parse]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/parse
[datetime-book]: http://php.net/manual/en/book.datetime.php
[datetime-class]: http://php.net/manual/en/class.datetime.php
[datetime-interface]: http://php.net/manual/en/class.datetimeinterface.php