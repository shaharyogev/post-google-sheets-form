# post-google-sheets-form
How to post a google sheets form with html form </br>
Hi first we will need the info from - https://medium.com/@dmccoy/how-to-submit-an-html-form-to-google-sheets-without-google-forms-b833952cc175

while running the original code on my website I got a ton of problems with it so after few versions, this is my final code.  You can see a live preview here - https://shahary.com

We need to creat a new google script - 
https://gist.github.com/davidmccoy/6183c5a847bb79a4aef257eb6238af53#file-google_script-gs




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

