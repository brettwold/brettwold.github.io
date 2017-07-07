---
layout: post
title:  "Save (Print) a PDF from an Android WebView"
subtitle: "A sneaky hack to save a PDF from any WebView"
date:   2017-04-06
categories: android PDF
tags: android PDF
author: Brett Cherrington
---

<section class="blog-intro">
<div class="col-md-12">
	<p>This post shows a nice trick (hack?) that allows you to programmatically save a PDF from any web page shown in an Android WebView.</p>
  <p>No external libraries required!</p>
</div>
</section>

<div class="col-md-12">
  <p>So how is it done?</p>
  <p>Well turns out that any WebView supports printing and essentially all the following code does is intercept the
  print process and instead of printing capture the print result (which is a PDF) and save it to disk. (Yes, its that simple!)</p>
  <p>So to acheive this we're going to write a PdfPrint class. The code below shows how we will use our new PdfPrint class</p>
  <script src="https://gist.github.com/brettwold/e8dc5ff02ecb07aebf4c96e130caf65b.js"></script>
  <p>As you can see the code makes use of a method on the WebView `createPrintDocumentAdapter`. This method returns
  a `PrintDocumentAdapter` object which is what we will use in our implementation to capture the output. Essentially the
  PdfPrint class shown below acts a bit like the Android PrintManager service and extracts the PDF from the WebView. The
  WebView itself does all the hard work of converting the page to a PDF we just save the output.</p>
  <p>The PrintPdf class is constructed with some details about the page size (`PrintAttributes`) you want to output and the
  print method itself takes a directory and filename of where to output the file to.</p>
	<p><strong>Update 16/Jun/17: Notice that the class below must also be placed in the `android.print` package. This is because the call
	to `LayoutResultCallback` is package local in Android.</strong></p>
	<p><string>Update 7/Jul/17: Note a minor change here to set the page range when calling `onWrite`. This fixes an issue with newer WebView implementations.</strong></p>
  <script src="https://gist.github.com/brettwold/838c092329c486b6112c8ebe94c8007e.js"></script>
  <p>The PrintPdf class simply calls the `onLayout` and `onWrite` methods of the `PrintDocumentAdapter` in order passing
  in the values for the print attributes and the output file as a `ParcelFileDescriptor`.</p>
  <p>As you can see there are a few nulls passed into the methods above that would normally be populated by the Android
  PrintManager. I have tried this example on several Android devices with Chromium as the underlying default WebView and it
  works sucessfully. However, other WebView implementations may require these values to be set. The only other restriction
  worth mentioning is that the PrintPdf class has to live in a package called `android.print`. This is so we have package
  level access to the inner class `LayoutResultCallback`.</p>
  <h2>Summary</h2>
  <p>The examples above show how you can in only a few lines of code save (print) a PDF to a file.</p>
  <p>Have fun!</p>

</div>
