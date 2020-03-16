# Fulcrum Data Events Snippets
Real world examples of JavaScript code I use on my organization's Fulcrum account and apps, which aren't featured on the Data Events documentation examples.

## Locally store field value from one record and display it in a different field in a new record
In this example, the user inputs the drop-off location of the car keys in the text field name 'keys_location' the end of the record form 
The value is the stored locally as the variable 'key' and displayed in read-only field name 'keys_ro' at the top of the form upon opening a new record.

The functionallity is similar to the "Default to previous item" option, but transfers the value to a new field, rather than displays it in the same field. Since the app has a checklist like working order, I wanted to display the value anew.

Code adapted from the Conditionally persist values across records example at
http://developer.fulcrumapp.com/data-events/examples/conditionally-use-previous-value/

```sh
var storage = STORAGE();

// when saving the record, save the value to storage to use next time
ON('save-record', function(event) {
  storage.setItem('keys', $keys_location);
});

ON('new-record', function(event) {
  // when creating a new record, set the value of the text field to the last value //used
  SETVALUE('keys_ro', storage.getItem('keys'));
});
```
If the Keys Location field is a choice field, then:

```sh
storage.setItem('keys', CHOICEVALUE($keys_location));
```

## GLOBALLY store field value from one record and display it in a different field in a new record
In this example, I recreate the functionality of the above example, but have it work globally, for any user using any device to enter the app and add records. This functionality can only work with the API, so you'll need to set up an API token from

https://web.fulcrumapp.com/settings/api

The data event has 3 variables:

token - API token.

formid - The app id which one can find in the url of the app.

key-from-schema - the REQUEST function queries the json, which has gives every individual fiels its own key and values:
```
"8373": {
  "choice_values": ["Pillar"],
  "other_values": []
}
```
you'll have to find the correct field key, which you can do by symply writing the url in a browser search box and inspecting the result.

```sh
var token = 'my-token';
var formid = 'form-id'

ON('new-record', function (event) {
  var options = {
    url: 'https://api.fulcrumapp.com/api/v2/records.json?form_id=' + formid + '&per_page=1&token=' + token + '&newest_first=1'
  };

  REQUEST(options, function(error, response, body) {
    if (error) {
      ALERT('Error with request: ' + error);
    } 
    else {
      var data = JSON.parse(body);
      if (data) {
        SETVALUE('keys_ro', data.records[0].form_values['key-from-schema'].choice_values[0])
       
      }
    }
  });
});
```



## Time-Sensitive field requirement
In this example, several fields are set to required if more than 7 days have passed since the date value entered in the field 'placement_date', which represents the placement of a camera trap. At this time, I can't have the 'end_date' field as required since the camera needs to be in place for 10 full days. When the record is loaded in the field at a later date, it checks if the difference between the current date is more than 7 days from the 'placement_date', and sets the 'end_date', 'end_time' and 'camera_status' fields to required if true

```sh
ON('load-record', function (event) {
  var today = new Date();
  if (today >= DATEADD($placement_date, 7)) {
    SETREQUIRED('end_date', true);
    SETREQUIRED('end_time', true);
    SETREQUIRED('camera_status', true);
  }
});
```

## Populate child records in a repeatable section with values from the parent record
In this example the app reads the variable $plot_id - which defines the plot name - any time a new repeatable is created in the section called 'observations', and populates the field 'plot_id_obs' with the value.

If the field is a choice or other, you need to wrap the $field with the correct data handler, e.g: CHOICEVALUE($field).

```sh
ON('new-repeatable', 'observations', function(event){
  SETVALUE ('plot_id_obs', $plot_id);
});
```

## Auto-Increment Calculation Field for Counting Order of Repeatable in Repeatable Section
This example create an auto-increment value using $repeatable.length method. Thou improves it so:
$repeatable.length+1 to get the current repeatable, not the count of the previous ones. Furthermore, it wraps it in an IF statement for the first reapeatable added which has ann eror. The else statement places 1 as default.


```sh
if ($repeatable && $repeatable.length) {
  SETRESULT($repeatable.length + 1);
} else {
  SETRESULT(1);
}
```
