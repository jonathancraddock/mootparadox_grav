---
title: 'Form Test 1'
form:
    name: ajax-test-form
    action: /forms/form-test-1
    template: form-messages
    refresh_prevention: true
    fields:
        name:
            label: 'Your Name'
            type: text
    buttons:
        submit:
            type: submit
            value: Submit
    process:
        message: 'Thank you for your submission!'
---

### Form Test 1

Some sample text here