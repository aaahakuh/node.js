# node.js
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="https://ssl.gstatic.com/docs/script/css/add-ons.css">
  <script>
    // IMPORTANT: Replace the value for DEVELOPER_KEY with the API key obtained
    // from the Google Developers Console.
    var DEVELOPER_KEY = 'AIzaSyAbQydqE2BiBPqaQRLTojXgB5Nc5z90VSk';
    var DIALOG_DIMENSIONS = {width: 800, height: 200};
    var pickerApiLoaded = false;

    /**
     * Loads the Google Picker API.
     */
    function onApiLoad() {
      gapi.load('picker', {'callback': function() {
        pickerApiLoaded = true;
      }});
     }

    /**
     * Gets the user's OAuth 2.0 access token from the server-side script so that
     * it can be passed to Picker. This technique keeps Picker from needing to
     * show its own authorization dialog, but is only possible if the OAuth scope
     * that Picker needs is available in Apps Script. Otherwise, your Picker code
     * will need to declare its own OAuth scopes.
     */
    function getOAuthToken() {
      google.script.run.withSuccessHandler(createPicker)
          .withFailureHandler(showError).getOAuthToken();
    }

    /**
     * Creates a Picker that can access the user's spreadsheets. This function
     * uses advanced options to hide the Picker's left navigation panel and
     * default title bar.
     *
     * @param {string} token An OAuth 2.0 access token that lets Picker access the
     *     file type specified in the addView call.
     */
    function createPicker(token) {
      if (pickerApiLoaded && token) {
        var view = new google.picker.View(google.picker.ViewId.SPREADSHEETS);
        var picker = new google.picker.PickerBuilder()
            // Instruct Picker to display only spreadsheets in Drive. For other
            // views, see https://developers.google.com/picker/docs/#otherviews
            // .addView(google.picker.ViewId.SPREADSHEETS)
            .addView(view)
            .addView(new google.picker.DocsUploadView())
            // Hide the navigation panel so that Picker fills more of the dialog.
            .enableFeature(google.picker.Feature.MULTISELECT_ENABLED)
            // Hide the title bar since an Apps Script dialog already has a title.
            .hideTitleBar()
            .setOAuthToken(token)
            .setDeveloperKey(DEVELOPER_KEY)
            .setCallback(pickerCallback)
            .setOrigin(google.script.host.origin)
            // Instruct Picker to fill the dialog, minus 2 pixels for the border.
            .setSize(DIALOG_DIMENSIONS.width - 2,
                DIALOG_DIMENSIONS.height - 2)
            .build();
        picker.setVisible(true);
      } else {
        showError('Unable to load the file picker.');
      }
    }

    /**
     * A callback function that extracts the chosen document's metadata from the
     * response object. For details on the response object, see
     * https://developers.google.com/picker/docs/result
     *
     * @param {object} data The response object.
     */
    function pickerCallback(data) {
      var action = data[google.picker.Response.ACTION];
      if (action == google.picker.Action.PICKED) {
        var doc = data[google.picker.Response.DOCUMENTS][0];
        var id = doc[google.picker.Document.ID];
        var url = doc[google.picker.Document.URL];
        var title = doc[google.picker.Document.NAME];
        document.getElementById('result').innerHTML =
            '<b>You chose:</b><br>Name: <a href="' + url + '">' + title +
            '</a><br>ID: ' + id;
      } else if (action == google.picker.Action.CANCEL) {
        document.getElementById('result').innerHTML = 'Picker canceled.';
      }
    }

    /**
     * Displays an error message within the #result element.
     *
     * @param {string} message The error message to display.
     */
    function showError(message) {
      document.getElementById('result').innerHTML = 'Error: ' + message;
    }
  </script>
</head>
<body>
  <div>
    <button onclick='getOAuthToken()'>Select a file</button>
    <p id='result'></p>
  </div>
  <script src="https://apis.google.com/js/api.js?onload=onApiLoad"></script>
</body>
</html>
