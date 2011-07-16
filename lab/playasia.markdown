---
layout: listing
title: PlayAsia.com api for Python
---

## PlayAsia.com api for Python

[PlayAsia](http://play-asia.com) is an online retailer for entertainment products from Asia. The website sells import games, DVDs, music, CDs, gadgets, groceries, books, gaming console accessories, cables and toys. Play-Asia is based in Hong Kong, and caters to the Hong Kong and Asia-Pacific region, but also offers most of the products to international buyers.

The [Play-Asia.com API](https://github.com/semk/PlayAsia) is a set of functions that allow you to retrieve data programmatically using Python. You need to register at [PlayAsia](http://play-asia.com) for the api key and user id.

### How to use this api?

{% highlight python %}
from playasia import PlayAsia

p = PlayAsia(api_key='YOUR-PLAYASIA-API-KEY', user_id='YOUR_ID')
result = p.test(quick=1)
result = p.info(quick=0, pax='PAX0003120605', mask='pnl', fx='INR')
result = p.paxfrombarcode(quick=0, bc='4974365910211')
result = p.paxfrommcode(quick=0, code='BLAS-50230')
{% endhighlight %}

### List of available functions

#### test()

You can use this to test if your account is properly set up. Try it by using

{% highlight python %}
# it should return 1 on success.
result = p.test(quick=1)
# it should return an object.
result = p.test(quick=0)
if not int(result.status.error):
    print result.status.query
    print result.content.test
else:
    print result.status.errorstring
{% endhighlight %}

On success, it should simply return 1. Omit the quick=1 and you will get a `result` object with properties mentioned in the above code.

#### info()

This will return the information about item(s). mask needs to be set according to the mask list below. Setting a mask of pnl for instance will give you price, name and item URL (along with PAX which always gets returned). Combine several PAX with `,`.

{% highlight python %}
# info method
result = p.info(quick=0, pax='PAX0003120605', mask='pnl', fx='INR')
if not int(result.status.error):
    print result.status.query
    print result.status.pax
    print result.status.mask
    print result.status.fx
    print result.status.items
    for item in result.content.item:
        print item.pax
        print item.price
        print item.fxprice
        print item.name
        print item.url
else:
    print result.status.errorstring
{% endhighlight %}

* `pax` = Play-Asia item number of item to query.
* `mask` = Which part of an item info will be delivered. See mask list below.
* `fx` = Foreign currency as reference. Use p mask. (EUR HKD GPB AUD CHF NZD CAD THB TWD MYR SGD BRL KRW IDR PHP MXN AED JPY INR)

#### paxfrombarcode()

This will return any matching PAX for this barcode (JAN, UPC, EAN, ...). The return set can consist of several products, for instance preowned, used or damaged goods.

{% highlight python %}
# using paxfrombarcode
result = p.paxfrombarcode(quick=0, bc='4974365910211')
if not int(result.status.error):
    print result.status.query
    print result.status.items
    for item in result.content.item:
        print item.pax
        print item.bc
        print item.name
else:
    print result.status.errorstring
{% endhighlight %}

* `bc` = Barcode (JAN/EAN/UPC ...) to search for.

#### paxfrommcode()

This will return any matching PAX for this manufacturer code. The return set can consist of several products for obvious reasons.

{% highlight python %}
# using paxfrommcode
result = p.paxfrommcode(quick=0, code='BLAS-50230')
if not int(result.status.error):
    print result.status.query
    print result.status.bc
    print result.status.items
    for item in result.content.item:
        print item.pax
        print item.code
        print item.name
else:
    print result.status.errorstring
{% endhighlight %}

* `code` = Manufacturer Code to search for.

#### listing() (not available now)

This is similar to info, so be sure to study this first. Furthermore, listing allows you to get a list of several items at once. By default, listing will return items in order of descending release date (newer items first).

{% highlight python %}
# using listing
result = p.listing(quick=0, mask='pnl')
{% endhighlight %}

* `mask` = item information you want to retrieve
* `results` = number of items to return (limited to 500 items)
* `start` = start at item start to list items beyond number 500. 0 = Index 0 = first record. 1 = Index 1 = second record. If 0, the result header will include total_items which you can use to ascertain how many items are in the complete result set. However, this number is an approximate only.
* `onsale` = item is on sale (not necessarily in stock right now)
* `instock` = item is currently in stock and on sale
* `type` = specify the item type. See list below. Combine multiple with ,
* `compatible` = specify the compatibility of item, ie, platform. See compatibility list below. Combine multiple with `,`
* `genre` = specify the genre of item. See genre list below. Combine multiple with `,`
* `encoding` = specify the encoding (NTSC-J, PAL, Region Free) of item. See encoding list below. Combine multiple with ,
* `version` = specify the version (Asia, Japan, USA, etc ...) of item. See version list below. Combine multiple with `,`
* `skip_preowned` = Specify to include preowned items in the return set 0 or 1
* `keyword` = Specifies keyword that must match the product name in the return set a-z0-9\s Please only use together with at least one compatible.
