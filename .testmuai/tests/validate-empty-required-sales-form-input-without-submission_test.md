---
assurance:
  id: t-1
  base: sha256:2db5244a7e0b0bcd14742abf130d162bcecd3bfbadb0833be37cd1e2b8729c99
---
# Validate empty required sales-form input without submission

> Prove that leaving required sales-form input empty triggers inline validation on the sales form without completing a real lead submission.

## Step 1

Open https://www.twilio.com/en-us in a new browser session as an anonymous visitor.

## Step 2 @verifies ac-2

From the homepage, use a visible Contact sales control to open the sales contact page, then assert the sales form shows at least one interactive field ready for input.

## Step 3 @verifies ac-3, ac-5

On the sales contact form, focus a required field and leave it as an empty string by blurring it without entering any characters, then assert inline validation is shown for the field and the sales form remains visible on the sales contact page for correction.

## Step 4 @verifies ac-3, ac-5

On the same required field or another required field on the same form, enter a single space and blur the field, then assert inline validation is shown again and the sales form remains visible on the sales contact page for correction.
