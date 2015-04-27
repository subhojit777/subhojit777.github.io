---
layout: post
title: Import images using Drupal feeds module
date: '2013-04-23T11:38:00.003+05:30'
author: Subhojit Paul
tags:
- drupal
- feeds import
modified_time: '2013-04-23T11:40:43.677+05:30'
thumbnail: http://2.bp.blogspot.com/-jo1LhFM58ac/UXYKZ3UfXTI/AAAAAAAAAg4/w9_dsUH_4tE/s72-c/Selection_001.png
blogger_id: tag:blogger.com,1999:blog-6340784984471653280.post-8709561121829630966
blogger_orig_url: http://subhojitpaul.blogspot.com/2013/04/import-images-using-drupal-feeds-module.html
---

Importing text content using Drupal feeds module is easy. Importing images using [feeds](http://drupal.org/project/feeds) is easy too but it requires a bit work and it is not as straightforward as importing text content.

At first you have to copy image files on your server, you can copy them anywhere inside your website directory, but I prefer copying it to public files directory(`sites/default/files`).

After you have copied the images, you need to create a CSV importer. Inside that CSV importer we will specify the path of the image. Look at the feed importer:

[![](http://2.bp.blogspot.com/-jo1LhFM58ac/UXYKZ3UfXTI/AAAAAAAAAg4/w9_dsUH_4tE/s1600/Selection_001.png)](http://2.bp.blogspot.com/-jo1LhFM58ac/UXYKZ3UfXTI/AAAAAAAAAg4/w9_dsUH_4tE/s1600/Selection_001.png)

Product images is the important thing in this feed importer. Product images is an image field.

Here is the CSV importer:

```bash
"SKU";"Title";"Description";"Brief Description";"Taxonomy Terms";"Product Selection";"Product Images"

"IMPORTED-PRODUCT-1";"Imported product display one";"This is an imported product one. This is an imported product one.";"This is an imported product one";"Clothing";"IMPORTED-PRODUCT-1";"http://mysite.local/sites/default/files/_MG_4933-s.jpg"
```

If you have already uploaded the image to the server then you can import data using the CSV.

This was about importing images using feeds. Suppose, the field accepts multiple images and you want to import multiple images using the importer, then you can use [feeds tamper](http://drupal.org/project/feeds_tamper) module for this. Look at the feeds tamper configuration for the images field:

[![](http://1.bp.blogspot.com/-KKRdBkJBVPI/UXYihO1zaCI/AAAAAAAAAhE/RdpZ9nH7wY0/s1600/Selection_002.png)](http://1.bp.blogspot.com/-KKRdBkJBVPI/UXYihO1zaCI/AAAAAAAAAhE/RdpZ9nH7wY0/s1600/Selection_002.png)

[![](http://4.bp.blogspot.com/-2zfghhjXOs4/UXYihVCcO7I/AAAAAAAAAhM/6598d2BCV9U/s1600/Selection_003.png)](http://4.bp.blogspot.com/-2zfghhjXOs4/UXYihVCcO7I/AAAAAAAAAhM/6598d2BCV9U/s1600/Selection_003.png)

You can change the exploding character to anything else, but make sure the character is unique and is **not present in the URL of image**.

Here is the CSV importer for importing multiple images:

```bash
"SKU";"Title";"Description";"Brief Description";"Taxonomy Terms";"Product Selection";"Product Images"

"IMPORTED-PRODUCT-1";"Imported product display one";"This is an imported product one. This is an imported product one.";"This is an imported product one";"Clothing~Jackets";"IMPORTED-PRODUCT-1";"http://mysite.local/sites/default/files/compressed/_MG_4933-s.jpg~http://mysite.local/sites/default/files/compressed/_MG_4934-s.jpg"
```

I hope everything is clear. Good luck :)
