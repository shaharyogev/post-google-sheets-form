# post-google-sheets-form
How to post a google sheets form with html form </br>
Hi first we will need the info from - https://medium.com/@dmccoy/how-to-submit-an-html-form-to-google-sheets-without-google-forms-b833952cc175

while running the original code on my website I got a ton of problems with it so after few versions, this is my final code.  You can see a live preview here - https://shahary.com

We need to creat a new google script - 

// original from: http://mashe.hawksey.info/2014/07/google-sheets-as-a-database-insert-with-apps-script-using-postget-methods-with-ajax-example/
// original gist: https://gist.github.com/willpatera/ee41ae374d3c9839c2d6 

function doGet(e){
  return handleResponse(e);
}

//  Enter sheet name where data is to be written below
        var SHEET_NAME = "Sheet1";

var SCRIPT_PROP = PropertiesService.getScriptProperties(); // new property service

function handleResponse(e) {
  // shortly after my original solution Google announced the LockService[1]
  // this prevents concurrent access overwritting data
  // [1] http://googleappsdeveloper.blogspot.co.uk/2011/10/concurrency-and-google-apps-script.html
  // we want a public lock, one that locks for all invocations
  var lock = LockService.getPublicLock();
  lock.waitLock(30000);  // wait 30 seconds before conceding defeat.
  
  try {
    // next set where we write the data - you could write to multiple/alternate destinations
    var doc = SpreadsheetApp.openById(SCRIPT_PROP.getProperty("key"));
    var sheet = doc.getSheetByName(SHEET_NAME);
    
    // we'll assume header is in row 1 but you can override with header_row in GET/POST data
    var headRow = e.parameter.header_row || 1;
    var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
    var nextRow = sheet.getLastRow()+1; // get next row
    var row = []; 
    // loop through the header columns
    for (i in headers){
      if (headers[i] == "Timestamp"){ // special case if you include a 'Timestamp' column
        row.push(new Date());
      } else { // else use header name to get data
        row.push(e.parameter[headers[i]]);
      }
    }
    // more efficient to set values as [][] array than individually
    sheet.getRange(nextRow, 1, 1, row.length).setValues([row]);
    // return json success results
    return ContentService
          .createTextOutput(JSON.stringify({"result":"success", "row": nextRow}))
          .setMimeType(ContentService.MimeType.JSON);
  } catch(e){
    // if error return this
    return ContentService
          .createTextOutput(JSON.stringify({"result":"error", "error": e}))
          .setMimeType(ContentService.MimeType.JSON);
  } finally { //release lock
    lock.releaseLock();
  }
}

function setup() {
    var doc = SpreadsheetApp.getActiveSpreadsheet();
    SCRIPT_PROP.setProperty("key", doc.getId());
}



Write a simple form like this- 

<form id="test-form">
                    <div class="form-group">
                      <label for="full_name" name="full_name">Contact me if you like :)</label>
                      <input type="full_name" name="full_name" class="form-control" id="full_name" placeholder="Full Name">
                    </div>
                    <div class="form-group">
                      <input type="phone_number" name="phone_number" class="form-control" id="phone_number" aria-describedby="emailHelp" placeholder="Phone Number">
                      <small id="phone_number" class="form-text text-muted"></small>
                    </div>
                    <div class="form-group">
                      <input type="email" name="email" class="form-control" id="email" aria-describedby="emailHelp" placeholder="Enter email">
                      <small id="email" class="form-text text-muted thanks">I'll never share your info with anyone else.</small>
                    </div>
                    <!--<div class="form-check">
                      <input type="checkbox" class="form-check-input" id="exampleCheck1">
                      <label class="form-check-label" for="exampleCheck1">Check me out</label>
                    </div>-->
                    <div>
                    <button type="submit" id="submit"  class="btn btn-primary">Submit</button>
                    </div>
                  </form>

The second thing is to geet the - Serialize Object function-
https://stackoverflow.com/questions/8900587/jquery-serializeobject-is-not-a-function-only-in-firefox


<script>
        $(document).ready(function(){
          $.fn.serializeObject = function()
              {
               var o = {};
               var a = this.serializeArray();
               $.each(a, function() {
                   if (o[this.name]) {
                       if (!o[this.name].push) {
                           o[this.name] = [o[this.name]];
                       }
                       o[this.name].push(this.value || '');
                   } else {
                       o[this.name] = this.value || '';
                   }
               });
               return o;
              };
            var form = $('form#test-form'),
              url = 'https://script.google.com/macros/s/AKfycbzlYpYZEFCuxD8gwTwM0w-hgO5XRmBwEzDG3ZhlvYrW/exec';
              form.submit(function(e){
                e.preventDefault();
              var jqxhr = $.ajax({
                url: url,
                method: "GET",
                dataType: "json",
                data: form.serializeObject()
              });
                $(".thanks").html("Thank you for getting in touch!").css("font-size","4rem");
                $(".form-control").remove();
                $("#submit").remove();
              });
            });


        </script>

