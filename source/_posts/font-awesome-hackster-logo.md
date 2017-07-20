---
title: Adding Custom Icon to Font Awesome
date: 2017-07-16 08:51:11
tags:
  - web development
  - graphic design
  - inkscape
---


{% asset_img hackster-logo.png Hackster Logo%}


I updated the 'follow' buttons on [samkristoff.com](my website) this morning and noticed that [Font Awesome](http://fontawesome.io/), does not have a [Hackster.io](https://www.hackster.io) icon.  I opened a [github issue](https://github.com/FortAwesome/Font-Awesome/issues/11298) requesting the new icon, but I need the icon now.

### Acquiring Source Images 
I noticed two Hackster logos on my [profile page](https://www.hackster.io/samkristoff) - the round one in the upper left and the square 'favicon'.  

{% asset_img hackster-potential-icons.png Potential Hackster Icons%}

The round one is easy to grab, just right click and **Save Image As**.  The favicon is a little more tricky, but using the Chrome Developer tools it's easy to find.  Right click anywhere in the page and choose **Inspect**.  Make sure the  **Elements**  tab is active, then press **Ctrl+F** and search for 'favicon' and you'll quickly find the image url.  Paste the url in a new tab and you can save a copy of the square icon as well.

### PNG to SVG

Both images are .png's and the round icon has some extra text.  We need clean .svg's for the next step so [paint.net](https://www.getpaint.net/) (or your editor of choice) to crop out the 'hackster.io' text from the round image.  Next import the first .png into [inkscape](https://inkscape.org/en/) by dragging it into a new Inkscape window.  Make sure the icon is selected and choose **Path>>Trace Bitmap**.  Under **Multiple Scans** choose **Colors** and set **Scans** to 2.  Click **OK** to generate the vector version of the logo then change the **Fill** to **#444444FF**.

{% asset_img inkscape-trace-bitmap.png Inkscape - Trace Bitmap%}

*In the end the circle logo was a bit too small so I cut went with the 'H' without the circle and it looked much better'*

### Generate a Custom Font Awesome Library

{% asset_img icomoon-custom-font-awesome.png Inkscape - Icomoon Custom Font Awesome Library%}

Launch the [Icomoon App](https://icomoon.io/app) to customize the Font Awesome library.  Choose **Add Icons From Library** and select **Font Awesome**.  Click the hamburger menu in the upper right, choose **Import to Set**, and select your new .svg.  Click **Generate Font** in the lower right and download the new custom Font Awesome library.  Extract the .zip and rename all the files in **/font** to **fontawesome-webfont**, leaving the extensions unaltered.  Convert the .woff to a .woff2 using your [tool](https://everythingfonts.com/woff-to-woff2) of choice and add the .woff2 to the **/font** folder.

We need to add css for the new icon.  On the Icomoon download page, hover over the new icon and click **Get Code**.  Copy the **CSS** and change '.icon-' to '.fa'.  Paste the updated **CSS** it into your font-awesome.css file.  For example:

```CSS
.fa-hackster:before {
  content: "\e901";
}
```

*Note: If you have both font-awesome.css and font-awesome.min.css chances are the .min version is being used in your website, make sure to update both.*


### Success!

The circle 

{% asset_img new-hackster-icon.png Inkscape - Fancy New Hackster Icon%}

