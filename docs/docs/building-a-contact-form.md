---
title: Building a Contact Form
---

This guide covers how to create a contact form in a Gatsby site, along with an overview of some strategies for handling form data that has been submitted.

Gatsby is built on top of React. So anything that is possible with a React form is possible in Gatsby. Additional details about how to add forms to gatsby can be found in the [Adding Forms](/docs/adding-forms/) section.

## Creating an Accessible Form

Faulty forms are a common barrier to a website's accessibility, and can be especially problematic if you use a keyboard and screen reader to navigate the web. Forms should be clearly and intuitively organized into groups of related information, and each form field should be identified with a proper label.

More information on creating accessible forms can be found in [WebAIM's article](https://webaim.org/techniques/forms/) on the subject.

## Sending Form Data

When you submit a form, the corresponding data is typically sent to a server to be handled in some manner. More in-depth information on sending form data can be found [on MDN](https://developer.mozilla.org/en-US/docs/Learn/HTML/Forms/Sending_and_retrieving_form_data).

Each method detailed below will start with the following contact form:

```jsx:title=src/pages/contact.js
<form method="post" action="#">
  <label>
    Name
    <input type="text" name="name" id="name" />
  </label>
  <label>
    Email
    <input type="email" name="email" id="email" />
  </label>
  <label>
    Subject
    <input type="text" name="subject" id="subject" />
  </label>
  <label>
    Message
    <textarea name="message" id="message" rows="5" />
  </label>
  <button type="submit">Send</button>
  <input type="reset" value="Clear" />
</form>
```

## Form submission options in Gatsby

### Formspree

Formspree offers a generous free-tier service for handling form submissions on static sites. This makes it a great tool for having form submissions sent directly to an email address, with very little setup required.

Since Gatsby is built on top of React, you can use a [React Hook](https://reactjs.org/docs/hooks-intro.html) to submit directly to Formspree.

> 💡 You'll need the Formspree React library and React 16.8.0 or later to use `useForm`.
>
> 📦 `npm install react@^16.8.0 @formspree/react`


```jsx:title=src/pages/contact.js
import { useForm } from '@formspree/react'

export default function ContactForm() {
  const [state, handleSubmit] = useForm('{your-form-id}')
  if (state.succeeded) {
    return <div>Thank you for contacting me!</div>;
  }
  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="email">Email</label>
      <input id="email" type="email" name="email" />
      <label htmlFor="message">Message</label>
      <input id="message" type="text" name="message" />
      <button disabled={state.submitting}>Sign up</button>
    </form>
  )
}
```

If you want to programmatically deploy forms, you can do so with the Formspree CLI.

> 📦 `npm install -g @formspree/cli`

You'll need to set up a `formspree.json` file in your project root that contains 
```json:title=formspree.json
{
    "forms": {
      "signupForm": {
        "name": "Contact Form",
        "fields": {
          "email": { "type": "email", "required": true },
          "message": { "type": "text", "required": true }
        },
        "actions": [
          { "type": "email", "to": "hello@example.com" }
        ],
        "allowExtraFields": false
      }
    }
  }
  ```

Now you need to deploy your configuration (or add it as part of your deploy command on productions sites)

  > `formspree deploy -k <your-deploy-key>`

To pass your project ID, you can use the [gatsby-browser.js](/docs/api-files-gatsby-browser/) file to wrap the root elements.

```jsx:title=gatsby-browser.js
import React from "react"

import { FormspreeProvider } from "@formspree/react"

export const wrapRootElement = ({ element }) => (
    <FormspreeProvider project={"{your-project-id}"}>{element}</FormspreeProvider>
  )
```

You can find more information on the registration process or setup [on their website](https://formspree.io/).

### Getform

Getform is a form backend platform which offers a free-plan for handling form submissions on static sites. Begin by creating a form on your Gatsby site that you can receive submissions from. When creating the form, direct the HTTP POST method to the Getform, by placing the `name` attributes for the fields you want to make visible. (name, email, message etc.)

```jsx:title=src/pages/contact.js
<form method="post" action="https://getform.io/{your-unique-getform-endpoint}">
  ...
  <label>
    Email
    <input type="email" name="email" />
  </label>
  <label>
    Name
    <input type="text" name="name" />
  </label>
  <label>
    Message
    <input type="text" name="message" />
  </label>
  ...
</form>
```

Once you've made the code changes to your form, you can head over to the contact page on your site and start submitting data to the form. The submissions will then be visible on the Getform dashboard. You can add multiple email addresses to receive email notifications for the forms created, as well as manipulate the data you see on Getform using Zapier and Webhooks options that are offered.

You can find more info on the registration process and form setup on the [Getform website](https://getform.io/) and find code examples (AJAX, reCAPTCHA etc) on their [CodePen](https://codepen.io/getform).

### Netlify

If you're hosting your site with Netlify, you gain access to their excellent [form handling feature](https://www.netlify.com/docs/form-handling/).

Setting this up only involves adding a few form attributes:

```diff:title=src/pages/contact.js
- <form method="post" action="#">
+ <form method="post" netlify-honeypot="bot-field" data-netlify="true" name="contact">
+   <input type="hidden" name="bot-field" />
+   <input type="hidden" name="form-name" value="contact" />
  ...
```

Now, all submissions to your form will appear in the Forms tab of your site dashboard. By adding the form attribute `netlify-honeypot="bot-field"` and a corresponding hidden input, Netlify will know to quietly reject any spam submissions you may receive.

More information on Netlify Forms can be found [on their website](https://www.netlify.com/docs/form-handling/).

### Run your own server

If your form data requires a significant amount of business logic to handle, creating your own service might make the most sense. The most popular solution to this is writing an HTTP server - this can be done in many languages including PHP, Ruby, GoLang, or in our case Node.js with [Express](https://expressjs.com/).

An initial implementation of a server using express, body-parser, and nodemailer may look like this:

```javascript:title=handleForm.js
const bodyParser = require("body-parser")
const express = require("express")
const nodemailer = require("nodemailer")

const app = express()
app.use(bodyParser.urlencoded())

const contactAddress = "hey@yourwebsite.com"

const mailer = nodemailer.createTransport({
  service: "Gmail",
  auth: {
    user: process.env.production.GMAIL_ADDRESS,
    pass: process.env.production.GMAIL_PASSWORD,
  },
})

app.post("/contact", function (req, res) {
  mailer.sendMail(
    {
      from: req.body.from,
      to: [contactAddress],
      subject: req.body.subject || "[No subject]",
      html: req.body.message || "[No message]",
    },
    function (err, info) {
      if (err) return res.status(500).send(err)
      res.json({ success: true })
    }
  )
})

app.listen(3000)
```

This initial implementation listens for POST requests to `/contact`, and sends you an email with the submitted form data. You can deploy this server with services such as [Vercel](https://vercel.com/home).

Once deployed, note the url of the deployment (something like `my-project-abcd123.vercel.app`), and use it as your form action:

```jsx:title=src/pages/contact.js
<form method="post" action="my-project-abcd123.vercel.app/contact">
  ...
</form>
```

Now, all subsequent form submissions will be sent to your email address!

For an in-depth guide on running your own mail server, you can refer to [this awesome guide by DataFire](https://medium.com/datafire-io/simple-backends-four-ways-to-implement-a-contact-us-form-on-a-static-website-10fc430984a4).

## Other resources

If you have any issues or if you want to learn more about implementing your own contact form in Gatsby, check out this tutorial from Scott Tolinski:

https://youtu.be/hF7xJhzrr9s
