# NextGEN plugin bug report #

This is to document a bug in the NextGEN WordPress plugin. The problem occurs when inserting a single picture from NextGEN gallery in post.

## Bug Description ##

The bug is seen when inserting one NextGEN gallery image in a post when there are more than 10 images in a NextGEN gallery. In this case the pictures are paged and some images are listed on multiple pages and some images are not listed on any of the pages.

## Steps to reproduce bug ##
Note: The versions are when the bug report was submitted to NextGen. Everything only applies to the free version of the plugin as I do not have access to the paid version to check it.
1. Fresh install of WordPress v4.9.4
1. Fresh install of free NextGEN Gallery v2.2.46. I also see the problem with the Beta v3.0.0.107
1. Create NextGEN gallery with 16 images (If the images are sequentially numbered it makes the issue easy to see)
1. Edit the welcome post
1. Select Add Media button, select NextGEN Gallery link, select the gallery which was created
1. In the picture page 1 is at the top and page 2 below. Note the images which are on pages 1 and 2: some images are listed twice and some not at all

## What and where the problem is ##
I tracked the problem down to this file: wp-content/plugins/NextGEN-galley/products/photocrati_NextGEN/modules/ngglegacy/media-upload.php and line 125:
```
$picarray = $wpdb->get_col("SELECT pid FROM $wpdb->nggpictures WHERE galleryid = '$galleryID' AND exclude != 1 ORDER BY {$ngg->options['galSort']} {$ngg->options['galSortDir']} LIMIT $start, 10 ");
```
Using xdebug I captured the $picarray for pages 1 and 2 for a gallery of 18 pictures with picture IDs 940-957 with the following results:

array | page 1 | page 2
----- | ------ | ------
0  |  957  |  947
1  |  949  |  946
2  |  948  |  945
3  |  947  |  944
4  |  946  |  943
5  |  945  |  942
6  |  944  |  941
7  |  943  |  953
8  |  942  |
9  |  941  |

This matches picture selection I see on pages 1 and 2.

## My implementation of a fix ##
For my impletation of a fix I read all of the gallery into an array then select the portion of the array needed for each page. While not ideal particularly for large galleries it does fix the problem.
```
$picarray = $wpdb->get_col("SELECT pid FROM $wpdb->nggpictures WHERE galleryid = '$galleryID' AND exclude != 1 ORDER BY {$ngg->options['galSort']} {$ngg->options['galSortDir']} LIMIT $start, 10 ");
```

## Bug Status at NextGEN ##
As of 8/27/2018 NextGEN has a test version of the plugin which fixes the problem but it has not yet been released.



