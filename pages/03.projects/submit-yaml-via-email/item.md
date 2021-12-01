---
title: 'Submit YAML via Email'
visible: false
taxonomy:
    category:
        - coding
        - projects
    tag:
        - javascript
        - jitbit
        - yaml
        - email
        - nodered
    projects-cat:
        - web
date: '31-10-2021 15:00'
---

!!!! Test to see whether it's possible to copy/paste straight from a GitHub markdown file.

^- *note to self, it breaks the image links, but works if replaced with githubusercontent raw links.*

===

**Experiment to pull a block of YAML data from an email, and write it to custom fields.**

Decided to go with a 'front-matter' style of syntax, placing the YAML between a header `---` and a footer `---`. Strict front-matter should begin on line one (I believe) but in this case it makes no difference. Also, the Jitbit API returns the ticket Body as HTML, and hence the regex/replace to strip out any tags.

```javascript
var emailBody = msg.payload.Body;
var emailYaml = emailBody.replace(/<(.|\n)*?>/g, '');

var starts = emailYaml.indexOf("---")+3;
var endsAt = emailYaml.indexOf("---", emailYaml.indexOf("---") + 1);

if (endsAt-starts<3) { msg.payload = "yaml : null"; } else
  { msg.payload = emailYaml.substring(starts,endsAt); }

return msg;
```
^- *initial test script to prove the `indexOf` combo!*

![](https://raw.githubusercontent.com/jonathancraddock/Jitbit-Custom/9a080ab923b42a99ad7cfcd7736c8ae2a40998f6/screencap/jitbit-post-yaml.png)

Jitbit automation rule triggers an HTTP POST when a new ticket is created in a specified category.

![](https://raw.githubusercontent.com/jonathancraddock/Jitbit-Custom/84fba9338730ef51c56dfde7c988fdc8f03cf94d/screencap/nodered-yaml-to-custom.png)

Grab the ticket ID and extract the block of YAML.

```javascript
// Get the ID and Body
msg.ticketId = msg.payload.ticketId;
var emailBody = msg.payload.emailBody;

// Strip HTML tags
var emailYaml = emailBody.replace(/<(.|\n)*?>/g, '');

// Write YAML to payload
var starts = emailYaml.indexOf("---")+3;
var endsAt = emailYaml.indexOf("---", emailYaml.indexOf("---") + 1);
if (endsAt-starts<3) { msg.payload = "yaml : null"; } else
  { msg.payload = emailYaml.substring(starts,endsAt); }

// Set the authorisation header
msg.headers = {};
msg.headers.Authorization = 'Bearer xxxxxxxxxxxx';

return msg;
```

> There's no meaningful validation here, unless it's missing altogether, or if the header/footer are incorrectly formed. (In which case you get `yaml : null`.)

Using the NodeRED YAML-node to turn that into a JSON object makes it easy to pick up the values required for the custom field data.

![](https://raw.githubusercontent.com/jonathancraddock/Jitbit-Custom/4f9674d78a98121728b1e8d940b43d7b0aee7436/screencap/yaml-to-json.png)

```javascript
// Set values
var colour = "&cf4xxx1="+msg.payload.colour;
var speed = "&cf4xxx2="+msg.payload.speed;
var pin = "&cf4xxx3="+msg.payload.pin;

//favourite colour = 4xxx1
//maximum speed    = 4xxx2
//debit card pin   = 4xxx3

// Values
var customFields = colour+speed+pin;

// Set the URL (SetCustomFields)
msg.url = "https://your-domain.jitbit.com/helpdesk/api/SetCustomFields?TicketId="+msg.ticketId+customFields;

return msg;
```

In this first test case, the email was sent as follows:

![](https://raw.githubusercontent.com/jonathancraddock/Jitbit-Custom/f7568ae69943477344d35a40c8cbec7987950034/screencap/yaml-email-example.png)

And the fields are populated almost immediately:

![](https://raw.githubusercontent.com/jonathancraddock/Jitbit-Custom/b15694b179c750294337e550294e5f679dbd2964/screencap/yaml-custom-fields-filled.png)

A niche requirement perhaps, but could be handy if there's a requirement to accept data from a 3rd party who can't directly call the API, or you might be able to use it to accept customised data via an external web-form; perhaps a security requirement is that it literally can't interact with the API.