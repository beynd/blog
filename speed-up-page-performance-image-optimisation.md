{
  "author": "Shane Neubauer",
  "photo": "https://cdn.gobeyond.so/displaypic/KxhnrzJ0ozOeB7TN3hLz5Fj1eLM2.jpg",
  "author_profile": "https://beyond.so/shane",
  "date": "2021-06-08T08:58:24.020Z",
  "title": "How To Improve Page Performance With Image Optimisation",
  "description": "How we used image optimisation to improve the user experience.",
  "image": "https://images.unsplash.com/photo-1530345586132-0fbc15e9ae5b?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=800&q=80"
}
---

Web page performance can make or break the user experience.

Even the most beautiful and functional design can be ruined if it feels sluggish and unresponsive. [Google published a quantified impact study of poor performance](https://web.dev/why-speed-matters/) on business, and the bottom line is simple: **users won't stick around if your web page feels slow.**

The first performance optimisation we performed on Beyond's web app was to improve images. For an image-heavy page, with user-uploaded images, it's often possible to have multiple megabytes of download, just to view the static page. Many megabytes for one page is way too large, and can result in a really poor experience for many users.

User-uploaded images are largely uncontrollable by us. The user decides which file to upload, and it could be anything from a few kilobytes all the way up to multiple megabytes. With ten or more images on a page (such as our [/discover](https://beyond.so/discover) page) this can result in a *very* unacceptable load time.

In one test, with my browser throttled to 'Fast 3G', a speed that is not uncommon for people in many locations, the page took up to *one minute* to finish visually rendering. **Unacceptable!**

# What we tried

We tried two approaches before settling on our current method:

1. [NextJS inbuilt image optimisation](https://nextjs.org/docs/basic-features/image-optimization)
2. ['Resize Images' Firebase Extension](https://firebase.google.com/products/extensions/storage-resize-images)

## NextJS Image Optimisation

NextJS has an inbuilt image optimisation feature. By importing and using the `next/image` library, which wraps the `<img>` HTML tag, you can take advantage of it.

This feature is nice — it allows for resizing, optimising, and serving the images without you having to do any heavy lifting. It also optimises the images at run time, not build time, so your builds will not get too lengthy.

**The problem we faced** with NextJS image optimisation was the lack of control over the actual element rendered on the page.

The `<Image>` tag passes any supplied properties through to the underlying tag ***except*** `style.` We were forced to adopt a different styling method just for the sake of using this feature. For us at Beyond, we use the [Chakra UI](https://chakra-ui.com/) kit along with its super nice theming, so introducing a totally new styling method just for this tag is not cool.

## Resize Images Firebase Extension

We're using Firebase Storage to store images, which works great. Coupled with Cloudflare, we have ourselves a nice little image repository, fronted by a global CDN.

So, when I found the turnkey extension for resizing and converting images, it felt like the obvious choice. I switched it on immediately and set up our app to use the freshly optimised images.

The set up is easy, and images will be automatically resized and converted to your favourite formats automatically each time a new image is uploaded. Simple!

Under the covers, it simply deploys a Firebase Function which is triggered by the 'object' `onFinalize` event.

**The problem we faced** with this extension was the very limited control over the image conversion and optimisation. Straight away we noticed that image quality was very poor. Yes, our page performance improved significantly with optimised images, but the appearance on the page was subpar, and very noticeable.

At Beyond we take pride in beautiful design, so the blurry images felt like a punch in the gut. Not acceptable.

# How we do it now

The solution we finally settled on is a custom Firebase Function, which behaves similarly to the official extension, but affords us a much greater level of control. We're using [the *sharp* library](https://www.npmjs.com/package/sharp) to do the conversion, and there are plenty of options to configure it how you need.

Here's how it works:

1. A user uploads a new image to the Firebase Storage bucket
2. The `onFinalize` event triggers our custom Firebase Function
3. The Firebase Function loads the image into memory, pipes it through *sharp*, and writes it out at multiple sizes back to Firebase Storage

![https://cdn.gobeyond.so/misc/blog/image-resizer.png](https://cdn.gobeyond.so/misc/blog/image-resizer.png)

Every image, no matter which size, resolution, or format, will be resized and stored in [WebP format](https://developers.google.com/speed/webp). We need different sizes for different components on the page, from small avatars at 24x24, to profile pictures at 400x400, and more.

The page will now render the perfect size for the use case, instead of downloading and scaling it in your browser. 

Using *sharp* we can convert to lossless WebP, which boasts a 26% smaller file size compared with PNG — another commonly used image format on the web. 

Because the function is custom built, we have total control over image optimisation. We can choose to support alpha channel (transparency) at a cost of 22% file size, or not. We can choose to use lossy compression and reduce file size even further.

The main thing: it's up to us. We have the control.

Sometimes it's great to get started with easy out-of-the box solutions, but you may find that you outgrow them quickly, just like we did. Our design is important to us, and we want our users to have a wonderful experience. Optimising our images properly has helped us on our mission to delight our users.

This was the first of many upcoming technical posts on our blog. Do you like reading this type of content? Want to read more? Share your feedback with us on [Twitter](https://twitter.com/sneub).
