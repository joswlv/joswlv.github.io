---
layout: post
title: 이미지 픽셀다루기
date: 2016-11-02 08:04
categories: js
---

# 이미지 픽셀값 뽑기

이미지를 로드 -> Canvas에 그리기 -> imgdata뽑기 -> 가지고 놀기!!


```javascript
var canvas;
var ctx;
var imgsrc;
function loadFile(event) {
    var output = document.getElementById('output');
    output.src = URL.createObjectURL(event.target.files[0]);
    imgsrc=output.src;
    $(function() {
        // drawing active image
        var image = new Image();
        // creating canvas object
        image.onload = function() {
            ctx.drawImage(image, 0, 0, image.width, image.height); // draw the image on the canvas
        };
        image.src = imgsrc;
        canvas = document.getElementById('panel');
        ctx = canvas.getContext('2d');
        $('#panel').mousemove(function(e) { // mouse move handler
            var canvasOffset = $(canvas).offset();
            var canvasX = Math.floor(e.pageX - canvasOffset.left);
            var canvasY = Math.floor(e.pageY - canvasOffset.top);
            var imageData = ctx.getImageData(canvasX, canvasY, 1, 1);
            var pixel = imageData.data;
            var pixelColor = "rgba(" + pixel[0] + ", " + pixel[1] + ", " + pixel[2] + ", " + pixel[3] + ")";
            $('#preview').css('backgroundColor', pixelColor);
        });
        $('#panel').click(function(e) { // mouse click handler
            var canvasOffset = $(canvas).offset();
            var canvasX = Math.floor(e.pageX - canvasOffset.left);
            var canvasY = Math.floor(e.pageY - canvasOffset.top);
            var imageData = ctx.getImageData(canvasX, canvasY, 1, 1);
            var pixel = imageData.data;
            $('#rVal').val(pixel[0]);
            $('#gVal').val(pixel[1]);
            $('#bVal').val(pixel[2]);
            $('#rgbVal').val(pixel[0] + ',' + pixel[1] + ',' + pixel[2]);
            $('#rgbaVal').val(pixel[0] + ',' + pixel[1] + ',' + pixel[2] + ',' + pixel[3]);
            var dColor = pixel[2] + 256 * pixel[1] + 65536 * pixel[0];
            $('#hexVal').val('#' + dColor.toString(16));
        });
    });
}
```

