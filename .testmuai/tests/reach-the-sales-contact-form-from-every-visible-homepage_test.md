---
mode: testing
url: https://www.twilio.com/en-us
headless: true
tags: [contact-sales, smoke, happy-path]
max_steps: 40
timeout: 300
assurance:
  id: t-3
  base: sha256:86db92fa9a73dd2551d1984870b5d7a3ab42ca97f3f74cb26e6fdc77a8c7400e
---
# Reach the sales contact form from every visible homepage Contact sales control

> Prove that the anonymous visitor can use a visible homepage Contact sales control to open the sales contact page and reach the rendered sales form.

## Step 1

Open https://www.twilio.com/en-us in a new browser session as an anonymous visitor.

## Step 2 @verifies ac-1

On the homepage, identify every visible Contact sales control and open each one in turn, returning to the homepage between checks as needed, then assert each control loads the sales contact page.

## Step 3 @verifies ac-2

On the sales contact page reached from the last Contact sales control checked, inspect the form area, then assert at least one interactive sales form field is visible.
