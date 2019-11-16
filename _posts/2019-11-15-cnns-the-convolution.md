---
layout: post
title: "CNNS: The convolution"
image: "https://i.stack.imgur.com/VC1TX.png"
categories:
 - Edge Case
tags:
 - deep learning
 - CNN
---

This post is the first of a series on deep learning, specifically Convolutional Neural Networks (CNNs), that consists of two theorical posts and one more practical, discussing a particular model. Let's start with the C of the initials CNN.

**Convolution** is a term used in mathematics to describing the operation on two functions that produces a third function expressing how the shape of one is modified by the other. I always remember a class in which the teacher explained this idea using a very simplistic example. Imagine that you are standing between two stages during a music festival: if you move closer to one or another, you will listen better to one song. If you stay in the middle, the signals are too mixed. This confusion is the convolution in action.

## Talking images

<div align="center">
<img src="https://media.giphy.com/media/8b5gDEqjO5BKM/giphy.gif">
</div>

First, the basics of images. To the computer, an image is nothing but a matrix of integers. When the picture is b&w, each integer between 0 and 255 represents the luminosity of the pixel. There are two ways of representing RGB figures. The first is a group of 3 integers that compose a pixel, with each of them representing red, blue, or green value. On another approach, the image with dimensions mxnx3 has one layer for each color.

In terms of image processing, convolute means executing multiple matrix multiplications between the matrix representing the image and the [mask](https://en.wikipedia.org/wiki/Kernel_(image_processing). Hence, we can apply several transformation filters, such as edge detection, blurring and sharpening effects.

## Types of image convolution

Although a conventional matrix multiplication requires that the number of columns of the 1st matrix must equal the number of rows of the 2nd matrix, the convolution process has no such condition. The kernel's dimensions don't need to match the image, as the convolution is not the same as a traditional product. Instead, the operation is repeated successively across the picture, as such as a sliding window moving over the image.

Considering that the shapes do not match, there are several types of convolution that result in distinct dimensions. The [convolve](https://docs.scipy.org/doc/scipy/reference/generated/scipy.signal.convolve2d.html) method in the scipy module presents three modes: *full*, in which the signals do not overlap completely at the end-points of the operation and boundary effects may occur; *same*, in which the outcome is the same size as the image; and *valid*, where the product consists only of those elements that do not rely on zero-padding.

## Talking math

<div align="center">
<img src="https://media.giphy.com/media/26xBI73gWquCBBCDe/giphy.gif">
</div>

The operation, as said before, is very similar to standard matrix multiplication. The differences begin with the size of the final matrix. When in the mode *full*, the resulting matrix C has the size *m* that combines the size of the kernel matrix K and the image B. The equation below shows:

$$
m = |K| + |B| - 1
$$

The values of $$C$$ are obtained by multiplying K and B.

$$
C(i, j) = \sum_{p=0}^{i+1} \sum_{q=0}^{j+1} K(p,q) B(i - p + 1, j - q + 1)
$$

where $$i$$ and $$j$$ are some positions in $$C$$, and the notation $$M(a, b)$$ denotes the element on the a-th row and b-th.

## Convolute and learn

The inspiration for using the idea of convolution to build a machine learning technique comes from a biological mechanism present in the brain of mammals. The visual cortex contains groups of neurons that are modulated by **receptive fields**, that "[can elicit neuronal responses when stimulated](https://en.wikipedia.org/wiki/Receptive_field)". Receptive fields overlap and repeat between neighboring cells, so they act as local filters over what they receive.

The [study](https://doi.org/10.1113%2Fjphysiol.1968.sp008455), that brings information about that region, divides the cells into two types: *simple*, that respond to specific patterns from within its receptive field; and *complex*, that have larger receptive fields and are insensitive to the exact position of the pattern in the area.

## Final thoughts

It is important (and also interesting!) to learn about all the steps in the computational learning process, and it is especially powerful to understand the reason why the model was built in a particular way. In the next posts, we will continue to see some theory and then dissect and train a ~~really~~ deep learning model.

## Further reading & references

[Image processing with OpenCV](https://docs.opencv.org/master/d2/d96/tutorial_py_table_of_contents_imgproc.html)

[CS123N](https://cs231n.github.io/convolutional-networks/)

[Activations of DCNNs are aligned with gamma band activty of human visual cortex](https://www.nature.com/articles/s42003-018-0110-y)

[Brain and the visual perception](https://books.google.com/books?id=8YrxWojxUA4C&pg=PA106)

[Receptive fields of single neurones in the cat's striate cortex](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC1363130)
