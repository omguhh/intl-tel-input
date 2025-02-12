# International Telephone Input [![Build Status](https://travis-ci.org/jackocnr/intl-tel-input.svg)](https://travis-ci.org/jackocnr/intl-tel-input)
A jQuery plugin for entering and validating international telephone numbers. It adds a flag dropdown to any input, detects the user's country, displays a relevant placeholder and provides formatting/validation methods.

<img src="https://raw.github.com/jackocnr/intl-tel-input/master/screenshot.png" width="424px" height="246px">

If you like it, please upvote on [Product Hunt](http://www.producthunt.com/posts/intl-tel-input)!

## Table of Contents

- [Demo and Examples](#demo-and-examples)
- [Features](#features)
- [Browser Compatibility](#browser-compatibility)
- [Getting Started](#getting-started)
- [Options](#options)
- [Public Methods](#public-methods)
- [Static Methods](#static-methods)
- [Events](#events)
- [Utilities Script](#utilities-script)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [Attributions](#attributions)


## Demo and Examples
You can view a live demo and some examples of how to use the various options here: http://jackocnr.com/intl-tel-input.html, or try it for yourself using the included demo.html.


## Features
* Automatically select the user's current country using an IP lookup
* Automatically set the input placeholder to an example number for the selected country
* Navigate the country dropdown by typing a country's name, or using up/down keys
* Handle phone number extensions
* The user types their national number and the plugin gives you the full standardized international number
* Full validation, including specific error types
* Retina flag icons
* Lots of initialisation options for customisation, as well as public methods for interaction


## Browser Compatibility
| Chrome | FF  | Safari | IE  | Chrome Android | Mobile Safari | IE Mob |
| :----: | :-: | :----: | :-: | :------------: | :-----------: | :----: |
|    ✓   |  ✓  |    ✓   |  8  |      ✓         |       ✓       |     ✓  |


## Getting Started
1. Download the [latest release](https://github.com/jackocnr/intl-tel-input/releases/latest), or better yet install it with [npm](https://www.npmjs.com/) or [Bower](http://bower.io)

2. Include the stylesheet
  ```html
  <link rel="stylesheet" href="path/to/intlTelInput.css">
  ```

3. Override the path to flags.png in your CSS
  ```css
  .iti-flag {background-image: url("path/to/flags.png");}
  ```
  _Update: you will now also need to override the path to flags@2x.png (for retina devices). The best way to do this is to copy the media query at the end of [intlTelInput.scss](https://github.com/jackocnr/intl-tel-input/blob/master/src/css/intlTelInput.scss) and update the path._

4. Add the plugin script and initialise it on your input element
  ```html
  <input type="tel" id="phone">

  <script src="//ajax.googleapis.com/ajax/libs/jquery/1.11.1/jquery.min.js"></script>
  <script src="path/to/intlTelInput.js"></script>
  <script>
    $("#phone").intlTelInput();
  </script>
  ```

5. **Recommended:** initialise the plugin with the `utilsScript` option to enable formatting/validation, and to allow you to extract full international numbers using `getNumber`.


## Options
Note: any options that take country codes should be [ISO 3166-1 alpha-2](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) codes  

**allowDropdown**  
Type: `Boolean` Default: `true`  
Whether or not to allow the dropdown. If disabled, there is no dropdown arrow, and the selected flag is not clickable. Also we display the selected flag on the right instead because it is just a marker of state.

**autoHideDialCode**  
Type: `Boolean` Default: `true`  
If there is just a dial code in the input: remove it on blur, and re-add it on focus. This is to prevent just a dial code getting submitted with the form. Requires `nationalMode` to be set to `false`.

**autoPlaceholder**  
Type: `Boolean` Default: `true`  
Add or remove input placeholder with an example number for the selected country. Requires the `utilsScript` option.

**customPlaceholder**  
Type: `Function` Default: `null`  
Change the placeholder generated by autoPlaceholder. Must return a string.

```js
customPlaceholder: function(selectedCountryPlaceholder, selectedCountryData) {
  return "e.g. " + selectedCountryPlaceholder;
}
```

**dropdownContainer**  
Type: `String` Default: `""`  
Specify the container for the country dropdown (use a jQuery selector e.g. `"body"`). This is useful when the input is within a scrolling element, or an element with `overflow: hidden`. Wherever you put the dropdown it will automatically close on the `window` scroll event to prevent positioning issues. If you want it to close when a different element is scrolled (such as the input's parent), simply listen for the that scroll event, and trigger `$(window).scroll()` e.g.

```js
$("#scrollingElement").scroll(function() {
  $(window).scroll();
});
```

**excludeCountries**  
Type: `Array` Default: `undefined`  
Don't display the countries you specify.

**geoIpLookup**  
Type: `Function` Default: `null`  
When setting `initialCountry` to `"auto"`, we need to use a special service to lookup the location data for the user. Write a custom method to get the country code. For example using the [ipinfo.io](http://ipinfo.io/) service:  
```js
geoIpLookup: function(callback) {
  $.get("http://ipinfo.io", function() {}, "jsonp").always(function(resp) {
    var countryCode = (resp && resp.country) ? resp.country : "";
    callback(countryCode);
  });
}
```
_Note that the callback must still be called in the event of an error, hence the use of `always` in this example._

**initialCountry**  
Type: `String` Default: `""`  
Set the default country by it's country code. You can also set it to `"auto"`, which will lookup the user's country based on their IP address - requires the `geoIpLookup` option - [see example](http://jackocnr.com/lib/intl-tel-input/examples/gen/default-country-ip.html). When instantiating the plugin, we now return a [deferred object](https://api.jquery.com/category/deferred-object/), so you can use `.done(callback)` to know when initialisation requests like this have finished. If you leave `initialCountry` blank, it will default to the first country in the list. _Note that if you choose to do the auto lookup, and you also happen to use the [js-cookie](https://github.com/js-cookie/js-cookie) plugin, it will store the loaded country code in a cookie for future use._

**nationalMode**  
Type: `Boolean` Default: `true`  
Allow users to enter national numbers (and not have to think about international dial codes). Formatting, validation and placeholders still work. Then you can use `getNumber` to extract a full international number - [see example](http://jackocnr.com/lib/intl-tel-input/examples/gen/national-mode.html). This option now defaults to `true`, and it is recommended that you leave it that way as it provides a better experience for the user.

**numberType**  
Type: `String` Default: `"MOBILE"`  
Specify one of the keys from the global enum `intlTelInputUtils.numberType` e.g. `"FIXED_LINE"` to tell the plugin you're expecting that type of number. Currently this is only used to set the placeholder to the right type of number.

**onlyCountries**  
Type: `Array` Default: `undefined`  
Display only the countries you specify - [see example](http://jackocnr.com/lib/intl-tel-input/examples/gen/only-countries-europe.html).

**preferredCountries**  
Type: `Array` Default: `["us", "gb"]`  
Specify the countries to appear at the top of the list.

**utilsScript**  
Type: `String` Default: `""` Example: `"build/js/utils.js"`  
Enable formatting/validation etc. by specifying the path to the included utils.js script (also available from [cdnjs.com](https://cdnjs.com/libraries/intl-tel-input)), which is fetched only when the page has finished loading (on window.load) to prevent blocking. When instantiating the plugin, we return a [deferred object](https://api.jquery.com/category/deferred-object/), so you can use `.done(callback)` to know when initialisation requests like this have finished. See [Utilities Script](#utilities-script) for more information. _Note that if you're lazy loading the plugin script itself (intlTelInput.js) this will not work and you will need to use the `loadUtils` method instead._


## Public Methods
**destroy**  
Remove the plugin from the input, and unbind any event listeners.  
```js
$("#phone").intlTelInput("destroy");
```

**getNumber**  
Get the current number in the given format (defaults to [E.164 standard](http://en.wikipedia.org/wiki/E.164)). The different formats are available in the enum `intlTelInputUtils.numberFormat` - taken from [here](https://github.com/googlei18n/libphonenumber/blob/master/javascript/i18n/phonenumbers/phonenumberutil.js#L883). Requires the `utilsScript` option. _Note that even if `nationalMode` is enabled, this can still return a full international number._  
```js
var intlNumber = $("#phone").intlTelInput("getNumber");
// or
var ntlNumber = $("#phone").intlTelInput("getNumber", intlTelInputUtils.numberFormat.NATIONAL);
```
Returns a string e.g. `"+17024181234"`

**getNumberType**  
Get the type (fixed-line/mobile/toll-free etc) of the current number. Requires the `utilsScript` option.  
```js
var numberType = $("#phone").intlTelInput("getNumberType");
```
Returns an integer, which you can match against the [various options](https://github.com/googlei18n/libphonenumber/blob/master/javascript/i18n/phonenumbers/phonenumberutil.js#L896) in the global enum `intlTelInputUtils.numberType` e.g.  
```js
if (numberType == intlTelInputUtils.numberType.MOBILE) {
    // is a mobile number
}
```
_Note that in the US there's no way to differentiate between fixed-line and mobile numbers, so instead it will return `FIXED_LINE_OR_MOBILE`._

**getSelectedCountryData**  
Get the country data for the currently selected flag.  
```js
var countryData = $("#phone").intlTelInput("getSelectedCountryData");
```
Returns something like this:
```js
{
  name: "Afghanistan (‫افغانستان‬‎)",
  iso2: "af",
  dialCode: "93"
}
```

**getValidationError**  
Get more information about a validation error. Requires the `utilsScript` option.  
```js
var error = $("#phone").intlTelInput("getValidationError");
```
Returns an integer, which you can match against the [various options](https://github.com/jackocnr/intl-tel-input/blob/master/src/js/utils.js#L175) in the global enum `intlTelInputUtils.validationError` e.g.  
```js
if (error == intlTelInputUtils.validationError.TOO_SHORT) {
    // the number is too short
}
```

**isValidNumber**  
Validate the current number - [see example](http://jackocnr.com/lib/intl-tel-input/examples/gen/is-valid-number.html). Expects an internationally formatted number (unless `nationalMode` is enabled). If validation fails, you can use `getValidationError` to get more information. Requires the `utilsScript` option. Also see `getNumberType` if you want to make sure the user enters a certain type of number e.g. a mobile number.  
```js
var isValid = $("#phone").intlTelInput("isValidNumber");
```
Returns: `true`/`false`

**setCountry**  
Change the country selection (e.g. when the user is entering their address).  
```js
$("#phone").intlTelInput("setCountry", "gb");
```

**setNumber**  
Insert a number, and update the selected flag accordingly. Optionally pass a `intlTelInputUtils.numberFormat` as the second argument if you want to specify national/international formatting (must be a valid number). _Note that by default, if `nationalMode` is enabled it will try to use national formatting._  
```js
$("#phone").intlTelInput("setNumber", "+44 7733 123 456");
```


## Static Methods
**getCountryData**  
Get all of the plugin's country data - either to re-use elsewhere e.g. to populate a country dropdown - [see example](http://jackocnr.com/lib/intl-tel-input/examples/gen/country-sync.html), or to modify - [see example](http://jackocnr.com/lib/intl-tel-input/examples/gen/modify-country-data.html). Note that any modifications must be done before initialising the plugin.  
```js
var countryData = $.fn.intlTelInput.getCountryData();
```
Returns an array of country objects:
```js
[{
  name: "Afghanistan (‫افغانستان‬‎)",
  iso2: "af",
  dialCode: "93"
}, ...]
```

**loadUtils**  
_Note: this is only needed if you're lazy loading the plugin script itself (intlTelInput.js). If not then just use the `utilsScript` option._  
Load the utils.js script (included in the lib directory) to enable formatting/validation etc. See [Utilities Script](#utilities-script) for more information.
```js
$.fn.intlTelInput.loadUtils("build/js/utils.js");
```


## Events
You can listen for the following events on the input.

**countrychange**  
This is triggered when the user selects a country from the dropdown.


## Utilities Script
A custom build of Google's [libphonenumber](http://libphonenumber.googlecode.com) which enables the following features:

* Formatting upon initialisation, as well as with `getNumber` and `setNumber`
* Validation with `isValidNumber`, `getNumberType` and `getValidationError` methods
* Placeholder set to an example number for the selected country - even specify the type of number (e.g. mobile) using the `numberType` option
* Extract the standardised (E.164) international number with `getNumber` even when using the `nationalMode` option

International number formatting/validation is hard (it varies by country/district, and we currently support ~230 countries). The only comprehensive solution I have found is libphonenumber, which I have precompiled into a single JavaScript file and included in the lib directory. Unfortunately even after minification it is still ~215KB, but if you use the `utilsScript` option then it will only fetch the script when the page has finished loading (to prevent blocking).

To recompile [Utilities Script](#utilities-script) see js-docs in top of [utils.js](src/js/utils.js).


## Troubleshooting
**Submitting the full international number when in nationalMode**  
If you're submitting the form using Ajax, simply use `getNumber` to get the number before sending it. If you're using the standard form POST method, you have two options. The easiest thing to do is simply update the input value using `getNumber` in a submit handler:  
```js
$("form").submit(function() {
  myInput.val(myInput.intlTelInput("getNumber"));
});
```
But this way the user will see their value change when they submit the form, which is weird. A better solution would be to update the value of a separate hidden input, and then read that POST variable on the server instead. See an example of this solution [here](http://jackocnr.com/lib/intl-tel-input/examples/gen/hidden-input.html).  

**Full width input**  
If you want your input to be full-width, you need to set the container to be the same i.e.
```css
.intl-tel-input {width: 100%;}
```

**Input margin**  
For the sake of alignment, the default CSS forces the input's vertical margin to `0px`. If you want vertical margin, you should add it to the container (with class `intl-tel-input`).

**Displaying error messages**  
If your error handling code inserts an error message before the `<input>` it will break the layout. Instead you must insert it before the container (with class `intl-tel-input`).

**Dropdown position**  
The dropdown should automatically appear above/below the input depending on the available space. For this to work properly, you must only initialise the plugin after the `<input>` has been added to the DOM.

**Placeholders**  
In order to get the automatic country-specific placeholders, simply omit the placeholder attribute on the `<input>`.

**Bootstrap input groups**  
Simply add this line to get [input groups](http://getbootstrap.com/components/#input-groups) working properly.
```css
.intl-tel-input {display: table-cell;}
```
_Note: there is currently [a bug](https://bugs.webkit.org/show_bug.cgi?id=141822) in Mobile Safari which causes a crash when you click the dropdown arrow (a CSS triangle) inside an input group. The simplest workaround is to remove the CSS triangle with this line: `.intl-tel-input .iti-flag .arrow {border: none;}`_

## Contributing
I'm very open to contributions, big and small! For instructions on contributing to a project on Github, see this guide: [Fork A Repo](https://help.github.com/articles/fork-a-repo).

You will need to install [npm](https://www.npmjs.org) to build the project. You will also need [evenizer](https://github.com/katapad/evenizer) (`npm install -g evenizer`) and [imagemagick](http://www.imagemagick.org/) to generate retina flag sprites. Then run `npm install` to install Grunt and other dependencies, then you should be good to run `grunt build` to build the project. At this point, the included demo.html should be working. You should make your changes in the `src` directory and be sure to run `grunt build` again before committing.

If when building, you get an error in the "exec:evenizer" task, you may need to temporarily increase the ulimit by running this command: `ulimit -S -n 2048`


## Attributions
* Flag images from [region-flags](https://github.com/behdad/region-flags)
* Original country data from mledoze's [World countries in JSON, CSV and XML](https://github.com/mledoze/countries)
* Formatting/validation/example number code from [libphonenumber](http://libphonenumber.googlecode.com)
* Lookup user's country using [ipinfo.io](http://ipinfo.io)
* Feature contributions are listed in the wiki: [Contributions](https://github.com/jackocnr/intl-tel-input/wiki/Contributions)


## Links
* List of [sites using intl-tel-input](https://github.com/jackocnr/intl-tel-input/wiki/Sites-using-intl-tel-input)
* List of [integrations with intl-tel-input](https://github.com/jackocnr/intl-tel-input/wiki/Integrations)
* Android native port: [IntlPhoneInput](https://github.com/Rimoto/IntlPhoneInput)
