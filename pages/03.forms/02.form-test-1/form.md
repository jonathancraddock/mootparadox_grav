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
        favcol:
            label: 'Favourite Colour'
            type: text
    buttons:
        submit:
            type: submit
            value: Submit
    process:
        message: 'Nothing has happened... just showing you a pointless message! ;-)'
---

### Form Test 1

Some sample text here

<div id="form-result"></div>

<script>
$(document).ready(function(){

    var form = $('#ajax-test-form');
    form.submit(function(e) {
        // prevent form submission
        e.preventDefault();

        // submit the form via Ajax
        $.ajax({
            url: form.attr('action'),
            type: form.attr('method'),
            dataType: 'html',
            data: form.serialize(),
            success: function(result) {
                // Inject the result in the HTML
                $('#form-result').html(result);
            }
        });
    });
});
</script>