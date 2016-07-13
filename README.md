# Stegnography-HTML-CSS-JS
Stegnography using javascript
HTML    Result
Edit on 
<h2>PART 3: Hide an Image in Another Image -- Steganography</h2>

<hr>
<table>
  <tr>
    <td>
      <a data-flickr-embed="true" href="https://www.flickr.com/photos/135024785@N06/21968595616/in/dateposted/" title="ChoppedImg"><img src="https://farm1.staticflickr.com/637/21968595616_c02ee38205_m.jpg" width="160" height="240" alt="ChoppedImg"></a>
      <script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
    </td>
    <td>
      <a data-flickr-embed="true" href="https://www.flickr.com/photos/135024785@N06/21806921238/in/photostream/" title="ChoppedMsg"><img src="https://farm1.staticflickr.com/740/21806921238_65208ce9f5_m.jpg" width="160" height="240" alt="ChoppedMsg"></a>
      <script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
    </td>
  </tr>
  <tr>
    <td>
      <a data-flickr-embed="true" href="https://www.flickr.com/photos/135024785@N06/21968595096/in/photostream/" title="Combined2Bits"><img src="https://farm1.staticflickr.com/620/21968595096_0b62a76002_m.jpg" width="160" height="240" alt="Combined2Bits"></a>
      <script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
    </td>
    <td>
      <a data-flickr-embed="true" href="https://www.flickr.com/photos/135024785@N06/21994777895/in/photostream/" title="Extracted2Bits"><img src="https://farm6.staticflickr.com/5712/21994777895_6f835367ab_m.jpg" width="160" height="240" alt="Extracted2Bits"></a>
      <script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
    </td>
  </tr>
</table>

<hr>
<h3>Algorithm Modification</h3>
<ul>
  <li>Add a parameter <b><em>numR</em></b> to the function <em>chop</em>, which denotes the number of lower bits to be replaced with 0s in order to hide another image.</li>
  <li>Add a parameter <b><em>numS</em></b> to the function <em>shift</em>, which denotes shifting an RGB value to the right by that number of bits.</li>
  <li>Note that <b><em>numR + numS = 8</em></b>.</li>
  <li>In order to extract the hidden image, in the function <em>extract</em>, we call another function named <b><em>lowerBits(value, numK)</em></b>, which we only keeps the lower <em>numK</em> bits of the value and then shifts it to the left by <em>8-numK</em>    bits.</li>
</ul>

<hr>
<h3>JavaScript Program</h3>
<pre>
// crop an image
function crop(img, width, height) {
    var cropped_img = new SimpleImage(width, height);
    for (var px of img.values()) {
        var x = px.getX();
        var y = px.getY();
        if (x < width && y < height) {
            cropped_img.setPixel(x, y, px);
        }
    }
    print(cropped_img);
    return cropped_img;
}
// replace the least-significant numR bits with 0s for the RGB values of each pixel
function chop(img, numR) {
    var choppedImg = new SimpleImage(img.getWidth(), img.getHeight());
    var tmp = Math.pow(2, numR);
    for (var pixel of choppedImg.values()) {
        var px = img.getPixel(pixel.getX(), pixel.getY());
        pixel.setRed(Math.floor(px.getRed() / tmp) * tmp);
        pixel.setGreen(Math.floor(px.getGreen() / tmp) * tmp);
        pixel.setBlue(Math.floor(px.getBlue() / tmp) * tmp);
    }
    return choppedImg;
}
// shift the RGB values of each pixel to the right by numS bits
function shift(img, numS) {
    var shiftedImg = new SimpleImage(img.getWidth(), img.getHeight());
    tmp = Math.pow(2, numS);
    for (var pixel of shiftedImg.values()) {
        var px = img.getPixel(pixel.getX(), pixel.getY());
        pixel.setRed(Math.floor(px.getRed() / tmp));
        pixel.setGreen(Math.floor(px.getGreen() / tmp));
        pixel.setBlue(Math.floor(px.getBlue() / tmp));
    }
    return shiftedImg;
}
// combine two images of the same size by adding their RGB values
function combine(image1, image2) {
    // create a blank image
    var new_image = new SimpleImage(image1.getWidth(), image1.getHeight());
    // set the RGB values of each pixel in the new image
    for (var pixel of new_image.values()) {
        var x = pixel.getX();
        var y = pixel.getY();
        var pixel1 = image1.getPixel(x, y);
        var pixel2 = image2.getPixel(x, y);
        pixel.setRed(pixel1.getRed() + pixel2.getRed());
        pixel.setGreen(pixel1.getGreen() + pixel2.getGreen());
        pixel.setBlue(pixel1.getBlue() + pixel2.getBlue());
    }
    return new_image;
}
// hide the image "image2" in the lower numH bits of the image "image2"
function hide(image1, image2, numH) {
    // crop the two images so that they are of the same size
    if (image1.getWidth() != image2.getWidth() || image1.getHeight() != image2.getHeight()) {
        var cropWidth = Math.min(image1.getWidth(), image2.getWidth());
        var cropHeight = Math.min(image1.getHeight(), image2.getHeight());
        image1 = crop(image1, cropWidth, cropHeight);
        image2 = crop(image2, cropWidth, cropHeight);
    }
    choppedImg = chop(image1, numH);
    shiftedImg = shift(image2, 8-numH);
    return combine(choppedImg, shiftedImg);
}
// extract the lower numK bits of a value
function lowerBits(value, numK) {
    return (value % Math.pow(2, numK)) * Math.pow(2, 8 - numK);
}
// extract the hidden image in the lower numK bits of the input image
function extract(image, numK) {
    var hiddenImage = new SimpleImage(image.getWidth(), image.getHeight());
    for (var pixel of hiddenImage.values()) {
        var px = image.getPixel(pixel.getX(), pixel.getY());
        pixel.setRed(lowerBits(px.getRed(), numK));
        pixel.setGreen(lowerBits(px.getGreen(), numK));
        pixel.setBlue(lowerBits(px.getBlue(), numK));
    }
    return hiddenImage;
}

var pic = new SimpleImage("astrachan.jpg");
var msg = new SimpleImage("MessageCSEveryone.jpg");
var combinedImg = hide(pic, msg, 4);
print(combinedImg);
var extractedImg = extract(combinedImg, 4);
print(extractedImg);

combinedImg = hide(pic, msg, 2);
print(combinedImg);
extractedImg = extract(combinedImg, 2);
print(extractedImg);

combinedImg = new SimpleImage("DrewHiding2bitMessage.png");
extractedImg = extract(combinedImg, 2);
print(extractedImg);
</pre>

<hr>
<h3>Hidden Message</h3>
<a data-flickr-embed="true" href="https://www.flickr.com/photos/135024785@N06/21806920678/in/photostream/" title="HiddenMsg"><img src="https://farm1.staticflickr.com/722/21806920678_43dce2ea07_m.jpg" width="140" height="210" alt="HiddenMsg"></a>
<script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

<hr>
