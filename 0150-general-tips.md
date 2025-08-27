General Tips
============

## Landing page forms

Use below snipplet as a bookmarklet to fill out forms on our default landing pages. Clicking this on the Bookmarks bar, while on the landing page, will fill out the form with randomized data.

Please make sure to m

If the record is the production system, please make sure to change the status to "Rejected (Test)" after submitting the form.


```javascript
javascript: (function () {
  var admFormElement = document.querySelector("adm-form");
  var shadowRoot = admFormElement.shadowRoot;
  var form = shadowRoot.querySelector("form");
  var randomString = Math.random().toString(36).substring(2, 7);
  var data = {
    phone: "888123123",
    email: "test" + randomString + "@example.com",
    firstname: "John" + randomString,
    lastname: "Doe",
    address: "Street 123",
    zip: "12345",
    city: "City",
    country: "PL",
    payment_type: "online",
  };
  for (var key in data) {
    if (data.hasOwnProperty(key)) {
      var field = form.elements.namedItem(key);
      field.value = data[key];
      field.dispatchEvent(new Event("input", { bubbles: true }));
      field.dispatchEvent(new Event("focus", { bubbles: true }));
    }
  }
})();
```
