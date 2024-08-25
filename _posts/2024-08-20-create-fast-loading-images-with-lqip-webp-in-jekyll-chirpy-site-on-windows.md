---
title: "Create Fast Loading Images with lqip webp in Jekyll Chripy Site on Windows"
description: Example how to create and running script on windows platform
date: 2024-08-20 00:00:00 +0700
comments: true
categories: [Web Dev, Jekyll]
tags: [jekyll, chripy, lqip, webp]
pin: true
mermaid: true
image:
  path: /assets/img/google-jekyll.webp
  lqip: data:image/webp;base64,UklGRlIAAABXRUJQVlA4IEYAAACQAwCdASoUAAwAPzmGulOvKSWisAgB4CcJaQAAUqcyf8QgjPAAAP3Sdf++BUC7J8gpEDjOZulNrffdweivL0HvR8CeQAAA
  alt: Build a task scheduler using bash.
---

This documentation will show you how to implement Low-Quality Image Placeholders (LQIP) in Jekyll site, enhancing user experience by displaying a low-resolution version of an image while the full-resolution image loads. You'll also learn how to prepare images specifically for the Chirpy theme and convert them to the efficient WebP format, ensuring optimal display and performance. These techniques can be applied to any Jekyll site, including those using the Chirpy theme, to significantly improve loading times and overall site performance.

## Prerequisites

To follow this tutorial, you will need the following open-source tools installed on your system:

- [WebP tools](https://developers.google.com/speed/webp/){:target="_blank"}
   - Developed by Google for efficient web image compression
   - Includes `cwebp` for encoding images to WebP format

- [ImageMagick](https://imagemagick.org/){:target="_blank"}
   - A comprehensive suite for image manipulation and conversion
   - Used for resizing, quality adjustment, and LQIP generation

Optional but recommended:
- [Canva](https://www.canva.com/){:target="_blank"} or similar visual editing tool
   - Useful for initial image resizing and composition

>
Step-by-step instructions for installing WebP tools and ImageMagick will be provided in the next section. Both tools are compatible with major operating systems (Windows, macOS, Linux). On this Tutorial just focus on Windows Platform.
{: .prompt-info }

## Benefits of LQIP

LQIP (Low-Quality Image Placeholder) is a technique where a low-resolution version of an image is encoded in base64 and placed in the front matter. This ensures a quick preview is displayed while the high-resolution image loads, improving the perceived loading time and user experience.

LQIP is beneficial for several reasons:
- **Improves User Experience**:
  - Provides faster perceived loading times by giving users a quick visual cue that an image is about to load.
  - Replaces blank spaces or broken image icons with a blurred or low-resolution version of the image, maintaining the visual integrity of the page.

- **Performance Benefits**:
  - Uses a small, low-resolution version of the image, which loads quickly and requires minimal data.
  - Particularly beneficial for users on slower connections or mobile devices.
  - Allows users to see content progressively load, making the website feel faster and more responsive.

- **SEO and Accessibility Improvements**:
  - Faster-loading pages can improve search engine rankings, as page speed is a factor in search algorithms.
  - Contributes to better performance metrics.
  - Increases the likelihood that users will stay on a page that loads quickly and smoothly, reducing bounce rates and potentially increasing user engagement and conversions.

- **Visual Appeal**:
  - Smooth transitions from a low-quality placeholder to a high-quality image provide a more visually appealing experience.
  - Adds a layer of polish to the website, making it look more professional and well-designed.

## Task 1: Prepare Your Image

- **Ensure Image Readiness**:
  - For the Chirpy theme, your image should have a resolution of `1200 x 630` pixels.
  - This resolution ensures the images look great and maintain a consistent aspect ratio.

- **Aspect Ratio Requirement**:
  - The aspect ratio should meet `1.91:1`.
  - If the aspect ratio does not meet this requirement, the image will be scaled and cropped.

- **Using Canva for Image Preparation**:
  - [Canva](https://www.canva.com/){:target="_blank"} is an excellent tool for resizing and cropping images effectively.
  - It provides a visual interface for precise control over cropping and composition.
  - Steps to prepare your image in Canva:
    1. Create a new design with custom dimensions of 1200x630 pixels.
    2. Upload your image and place it within the design.
    3. Adjust the image to fit the frame, ensuring important elements are visible.
    4. Export the image as a high-quality PNG or JPG.



