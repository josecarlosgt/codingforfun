# codingforfun
Coding for Fun

## Uploading Images to Google Drive with JavaScript

[Spreadsheet](https://docs.google.com/spreadsheets/d/1mbOKiW0b1hxQza26ogBCPrEdfQ2mf-JLH4iRr0xNftA/edit?usp=sharing)
[Directory with images](https://drive.google.com/drive/folders/14binR4PYA2ok8Po_uSngWlr6qnXKSVHK?usp=sharing)

### Front-end Code

```HTML
<!DOCTYPE>
<html>

        <head>
                <title>Image Upload</title>
                <meta charset="UTF-8">
        </head>


        <body>
                <h1>Coding for Fun</h1>
                <h2>Uploading Images to Google Drive using JavaScript</h2>
                
                <form action="https://script.google.com/macros/s/AKfycbyhZ-o9v4MOjrdhBWQwPFcz5pS80HKvjlCupqHkLJ47DIvEFEdq_vQiU-fhHnLA3_L8YQ/exec" method="POST">
                         <input type="file" id="imageInput" name="image"><br><br>
                        <input type="button" value="Upload Image" onclick="uploadImage()">
                </form>

        <script>
        function uploadImage() {
          const file = document.getElementById("imageInput").files[0];
          const reader = new FileReader();
        
          reader.onload = function () {
            const APPS_SCRIPT_URL = "https://script.google.com/macros/s/AKfycbwxrGZVie6lTGprHazcaXYrEGjRoqu2DNLw379OIYhdoXqt81KcwlMe_uQzG-pcyE6v/exec"
            const base64 = reader.result.split(',')[1]; // Remove "data:image/jpeg;base64,"
        
            // Send to Google Apps Script Web App
            fetch(APPS_SCRIPT_URL, {
              method: "POST",
              headers: {
                "Content-Type": "application/x-www-form-urlencoded"
              },
                body: "image=" + encodeURIComponent(base64)
            })
            .then(response => response.text())
            .then(result => alert(result))
            .catch(error => console.error("Upload failed:", error));
          };
        
          reader.readAsDataURL(file); // Triggers `onload` when done
        }
        </script>

        </body>
</html>
```

### Back-end Code (to be executed in Google App Script)

```JAVASCRIPT
function doPost(e) {
  try {
    const SHEET_ID = "1mbOKiW0b1hxQza26ogBCPrEdfQ2mf-JLH4iRr0xNftA";
    const sheet = SpreadsheetApp.openById(SHEET_ID).getActiveSheet();

    const imageData = e.parameter.image;
    if (!imageData) throw "No image data received.";

    const blob = Utilities.base64Decode(imageData);

    // Create a timestamp string to create unique file names for the image
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');

    // Define the filename with timestamp
    const filename = `uploaded-${timestamp}.jpg`;

    // Get or create the 'codingforfun' folder at root
    const folders = DriveApp.getFoldersByName("codingforfun");
    const folder = folders.hasNext() ? folders.next() : DriveApp.createFolder("codingforfun");

    // Create the image file in the folder
    const file = folder.createFile(Utilities.newBlob(blob, "image/jpeg", filename));

    sheet.appendRow([new Date(), file.getUrl()]);

    return ContentService.createTextOutput("Upload successful!");
  } catch (err) {
    return ContentService.createTextOutput("Error: " + err);
  }
}
```

