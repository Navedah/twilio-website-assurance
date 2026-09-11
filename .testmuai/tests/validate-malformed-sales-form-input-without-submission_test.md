---
assurance:
  id: t-2
  base: sha256:59f3152b0a7f06cedc69cc35bfed27ef8161cd90daed944efbe6eee1881d4338
---
# Validate malformed sales-form input without submission

> Prove that malformed input on a format-constrained sales-form field triggers inline validation on the sales form without completing a real lead submission.

## Step 1

Open https://www.twilio.com/en-us in a new browser session as an anonymous visitor.

## Step 2 @verifies ac-2

From the homepage, use a visible Contact sales control to open the sales contact page, then assert the sales form shows at least one interactive field ready for input.

## Step 3 @verifies ac-4, ac-6

On the sales contact form, locate a format-constrained field that the page validates, such as an email-address field when present, enter buyer.example.com and blur the field, then assert inline validation is shown and the sales form remains visible on the sales contact page for correction.

## Step 4 @verifies ac-4, ac-6

On the same format-constrained field, enter buyer@example.com. and blur the field, then assert inline validation is shown again and the sales form remains visible on the sales contact page for correction.
