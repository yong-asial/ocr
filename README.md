# OCR with Javascript

Optical Character Recognition (OCR) is a widespread technology to recognize text inside images. It is used to convert images containing text (typed, handwritten, or printed) into machine-readable text data. In this repo, we are creating a simple OCR application with Google Cloud Version API.

## What are we building?

Here, it is what we are going to build. It takes a picture with a mobile phone camera or pc webcam, displays it as the first picture, then draws bounding boxes around detected text in the second picture, and finally outputs all the text at the bottom.

## Get Cloud Vision API Key

To extract text information from an image, in this article, we are using Cloud Vision API provided by Google. First, you will need to create a project in Google Cloud, then enable billing, enable Cloud Vision API, and finally create an API key to access the Cloud Vision API.

The Cloud Vision API recognizes text in 100+ different languages and can be tested freely with the first 1000 requests. it is built for both recognizing sparse text in images (such as photos of business cards) and recognizing densely-spaced text in pictures of scanned documents.

## Start Project

- Run `npm install` to install project dependencies.
- Run `monaca preview` to preview the app.

## Tutorial

- [Medium Blog](https://medium.com/p/e7639099fba/edit)
