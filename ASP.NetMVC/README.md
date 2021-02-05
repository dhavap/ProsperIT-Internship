# C# Live Project
After completing my C# course, I worked on a 2 week sprint in a team of developers to complete stories from a live project. We used the ASP.NET MVC framework. We were creating an employee management portal for a construction company that would allow users to see their own schedules and share news related to their jobs. 

## Tech Used
This project was created using C#, the .Net Framework and Entity Framework. Azure Devops was utilised to manage the project. The IDE used was Visual Studio

## Features
### Forgot Password Implementation
In the login page, employees needed to be able to reset their password if they had forgotten it. I was tasked with implementing this. The employee first keys in the email address associated with their account in the forgot password form. I used Simple Mail Transfer Protocol(SMTP) to send a password reset email to aforementioned email address. If the email cannot be found in the database, a user-friendly error message is displayed informing the employee that the email does not exist and that they should register instead. 

```
AccountController.cs

// GET: /Account/ForgotPassword
        [AllowAnonymous]
        public ActionResult ForgotPassword()
        {
            return View();
        }


        // POST: /Account/ForgotPassword
        [HttpPost]
        [AllowAnonymous]
        [ValidateAntiForgeryToken]
        public async Task<ActionResult> ForgotPassword(ForgotPasswordViewModel model)
        {
            if (ModelState.IsValid)
            {
                var user = await UserManager.FindByEmailAsync(model.Email);
                if (user == null || await UserManager.IsEmailConfirmedAsync(user.Id)) //if user is not in database, this page displays
                {
                    return View("NoUserError");
                }
                //Send an email with this link
                string code = await UserManager.GeneratePasswordResetTokenAsync(user.Id);
                var callbackUrl = Url.Action("ResetPassword", "Account", new { userId = user.Id, code = code }, protocol:   Request.Url.Scheme);

                var toAddress = model.Email.ToString();
                var fromAddress = "liveprojectdummyacct@gmail.com";
                var message = "<b>Email: </b>" + model.Email + "<br>" + "<b>Message: Please reset your password by clicking</b><a href=\"" + callbackUrl + "\"><b>here</b></a>";//insert message

                SendEmails.SendEmail(toAddress, fromAddress, message);
                if (SendEmails.SendEmail(toAddress, fromAddress, message))
                {
                    ModelState.Clear();
                    ViewBag.Message = "An email has been sent to " + fromAddress + ". Please follow the instructions to reset your password.";
                }
                else
                {
                    ViewBag.Message = "Failed to deliver message to " + toAddress;
                }
                return RedirectToAction("ForgotPasswordConfirmation", "Account");
            }

            // If we got this far, something failed, redisplay form
            return View(model);
        }
        // GET: /Account/ForgotPasswordConfirmation
        [AllowAnonymous]
        public ActionResult ForgotPasswordConfirmation()
        {
            return View();
        }

        // GET: /Account/ResetPassword
        [AllowAnonymous]
        public ActionResult ResetPassword(string code)
        {
            return code == null ? View("Error") : View();
        }

        //
        // POST: /Account/ResetPassword
        [HttpPost]
        [AllowAnonymous]
        [ValidateAntiForgeryToken]
        public async Task<ActionResult> ResetPassword(ResetPasswordViewModel model)
        {
            if (!ModelState.IsValid)
            {
                return View(model);
            }
            var user = await UserManager.FindByEmailAsync(model.Email);
            if (user == null)
            {
                // Don't reveal that the user does not exist
                return RedirectToAction("NoUserError", "Account");
            }
            var result = await UserManager.ResetPasswordAsync(user.Id, model.Code, model.Password);
            if (result.Succeeded)
            {
                return RedirectToAction("ResetPasswordConfirmation", "Account");
            }
            AddErrors(result);
            return View(); //return view
        }

        //
        // GET: /Account/ResetPasswordConfirmation
        [AllowAnonymous]
        public ActionResult ResetPasswordConfirmation()
        {
            return View();
        }
```

```
SendEmails.cs

namespace ManagementPortal.Helpers
{
    public static class SendEmails
    {
        public static bool SendEmail(string toAddress, string fromAddress, string message)
        {
            using (MailMessage newMessage = new MailMessage())
            {
                newMessage.From = new MailAddress(fromAddress);
                newMessage.To.Add(new MailAddress(toAddress));
                newMessage.Subject = "Password Reset";
                newMessage.Body = message.ToString();
                newMessage.IsBodyHtml = true;
                try
                {
                    SmtpClient smtp = new SmtpClient();
                    smtp.Host = "smtp.gmail.com";
                    smtp.Port = 587;
                    smtp.Credentials = new NetworkCredential
                    ("liveprojectdummyacct@gmail.com", "ASDqwe123!");
                    smtp.EnableSsl = true;
                    smtp.Send(newMessage);
                    return true;
                }
                catch (SmtpFailedRecipientsException ex)
                {
                    foreach (SmtpFailedRecipientException t in ex.InnerExceptions)
                    {
                        var status = t.StatusCode;
                        if (status == SmtpStatusCode.MailboxBusy ||
                            status == SmtpStatusCode.MailboxUnavailable)
                        {
                            Thread.Sleep(3000);
                            //resend
                            SmtpClient smtp = new SmtpClient();
                            smtp.Host = "smtp.gmail.com";
                            smtp.Port = 587;
                            smtp.Credentials = new NetworkCredential
                            ("liveprojectdummyacct@gmail.com", "ASDqwe123!");
                            smtp.EnableSsl = true;
                            smtp.Send(newMessage);
                            return true;
                        }
                        else
                        {
                            return false;
                        }
                    }
                }
                catch (Exception ex)
                {
                    return false;
                }
                finally
                {
                    newMessage.Dispose();
                }
                return false;
            }
        }

        private static string RedirectToAction(string v1, string v2)
        {
            throw new NotImplementedException();
        }
    }
}
```

### Create News Modal
Employees were able to create news items but had to go to a separate create page to do so. My task was to create a News Item Modal that could be accessed from anywhere in the app. I created a modal that had the same fields as the Create Page for Company News. I then utilized jQuery to save the news item to the database. I created a link for this modal from the employee dashboard. 

```
Dashboard.cshtml

<h4>@{Html.RenderAction("_CreateNews", "CompanyNews");}</h4> 
```

```
CompanyNewsController.cs

//Partial view of CreateNews Modal
        public ActionResult _CreateNews()
        {
            return PartialView("_CreateNews");
        }

        // POST: CompanyNews/Create
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult _CreateNews([Bind(Include = "NewsId,DateStamp,Title,NewsItem,ExpirationDate")]
        CompanyNews companyNews)
        {
            if (ModelState.IsValid)
            {
                companyNews.DateStamp = DateTime.Now.ToString("MM/dd/yyyy hh:mm tt");
                db.CompanyNews.Add(companyNews);
                db.SaveChanges();
            }
            return PartialView();
        }
```

```
_CreateNews.cshtml

@using ManagementPortal.Models
@model CompanyNews

<!-- BUTTON THAT CREATES MODAL POP-UP -->
<h4 class="btn float-right text-dark" data-toggle="modal" data-target="#createNews" data-whatever="">Create News Item</h4>

@using (Html.BeginForm())
{
    @Html.AntiForgeryToken()
    <div class="modal fade bd-example-modal-lg" id="createNews" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
        <div class="modal-dialog modal-lg" role="document">
    <div class="modal-content">
    <div class="modal-header">
        <h5 class="modal-title col-lg" id="exampleModalLabel">Create News</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
            <span aria-hidden="true">&times;</span>
        </button>
    </div>

    <div class="">
        <form>
            <!--form input-->
            <div class="form-group">
                @Html.LabelFor(model => model.ExpirationDate, htmlAttributes: new { @class = "control-label col-lg" })
                <div class="col-lg">
                    @Html.EditorFor(model => model.ExpirationDate, new { htmlAttributes = new { @class = "form-control datepicker", required = "required", @readonly = "readonly", id = $"expDate_" } })
                    @Html.ValidationMessageFor(model => model.ExpirationDate, "", new { @class = "text-danger" })
                </div>
            </div>
            <div class="form-group">
                @Html.LabelFor(model => model.Title, htmlAttributes: new { @class = "control-label col-lg" })
                <div class="col-lg">
                    @Html.EditorFor(model => model.Title, new { htmlAttributes = new { @class = "form-control", required = "required" } })
                    @Html.ValidationMessageFor(model => model.Title, "", new { @class = "text-danger" })
                </div>
            </div>
            <div class="form-group">
                @Html.LabelFor(model => model.NewsItem, htmlAttributes: new { @class = "control-label col-lg" })
                <div class="col-lg">
                    @Html.EditorFor(model => model.NewsItem, new { htmlAttributes = new { @class = "form-control", required = "required", name = "the-textarea", id = "the-textarea", maxlength = "300" } })
                    @Html.ValidationMessageFor(model => model.NewsItem, "", new { @class = "text-danger" })
                    <div id="the-count">
                        <span id="current">0</span>
                        <span id="maximum">/300</span>
                    </div>
                </div>
            </div>
            <div class="modal-footer">
                <!--form buttons-->
                <input type="submit" value="Create" class="btn-base btn-w-100px" id="submit" />
                <button type="button" class="btn-base btn-w-100px" data-dismiss="modal">Close</button>
            </div>
        </form>

    </div>

    </div>
        </div>
            </div>
}


@section Scripts {
    @Scripts.Render("~/bundles/jqueryval")
    @Scripts.Render("~/bundles/jqueryui")

    <script type="text/javascript">

        $("#expDate_").datepicker({
            defaultDate: "+1d",
            changeMonth: false,
            numberOfMonths: 1,
            minDate: "+1d"
        });
        $('textarea').keyup(function () {
            var characterCount = $(this).val().length,
                current = $('#current'),
                maximum = $('#maximum'),
                theCount = $('#the-count');

            current.text(characterCount);
        });
    </script>
}
```


### Navbar scroll animation
My task was to make the horizontal navbar sticky. I was also asked to animate the navbar any way I see fit on scroll. This involved changing the styling on the CSS page as well as introducing some Javascript. 
<br>I changed the position of the navbar to absolute and moved it to the left top most corner. There were many nested elements in this navbar so any change tended to cause other unintended changes as well. Using developer tools, I was able to experiment and figure out how to solve the problems before actually changing the code. This proved to be far more efficient that changing the code on my branch and refreshing the page each time. 

```
site.css

/*===== FIX NAVBAR TO TOP LEFT CORNER =====*/
header {
    width: 100%;
    padding: 0px;
}
#head-nav{
     position: fixed;
     padding: 0;
     height: 110px;
     top: 0;
     left: 0;
     width: 100%;
     display: grid;
 }
 
 #menu {
     z-index: 101;
    }
    
#left-half {
     position: absolute;
     left: 0px;
     float: left;
     grid-column: 1;
     width: 50%;
     top: 20px;
 }
 
 /*===== ANIMATED STICKY HEADER =====*/
    .changeColorFirst {background: rgba(106, 196, 217, .4);}
    .changeColorSecond {background: rgba(106, 196, 217, .7);}
    .changeColorThird {background: rgba(106, 196, 217, 1);}
    header{transition: 0.3s;}
    
 /*===== RESPONSIVE HEADER =====*/
@media screen and (max-width: 770px) {
    #right-half {left: 40%;}
    #left-half {padding-left: 18%;}
}
@media screen and (min-width: 771px) and (max-width: 990px) {
    #right-half {left: 50%;}
    #left-half {padding-left: 15%;}
}
@media screen and (min-width: 991px) and (max-width: 1199px) {
    #right-half{left: 60%;}
    #left-half{padding-left: 13%;}
}
@media screen and (min-width: 1200px) {
    #right-half {margin-right: 50%;left: 68%;}
    #left-half {padding-left: 10%;}
}   
```

```
site.js

// CHANGE NAVBAR COLOR ON SCROLL
$(function () {
        $(window).scroll(function () {
            var headerNav = document.getElementById('head-nav');
            if ($(this).scrollTop() > 10) {
                headerNav.classList.add('changeColorFirst');
                headerNav.classList.remove('changeColorSecond', 'changeColorThird');
            }
            if ($(this).scrollTop() > 580) {
                headerNav.classList.remove('changeColorFirst', 'changeColorThird');
                headerNav.classList.add('changeColorSecond');
            }
            if ($(this).scrollTop() > 1000) {
                headerNav.classList.remove('changeColorSecond');
                headerNav.classList.add('changeColorThird');
            }
            if ($(this).scrollTop() <= 10) {
                headerNav.classList.remove('changeColorFirst', 'changeColorSecond', 'changeColorThird');
            }
        });
    });
```

###Button styling issues
Some of the buttons were not displaying the required styling. I used the page inspect tool on my browser to figure out why the styling was not being applied and realised that it was a matter of increasing specificity. In some cases, I added a class, in others, I had to add an Id to the button. This allowed me to override the current styling
<br>

```
/*============== UNREGISTERED USER PAGE STYLING ================*/ 
#createBtn a {
    text-decoration: none;
    color: white
}

/*========= fix btn style issues on createUserRequest/Confirmationcode page =======*/
.buttonStyle a {
    text-decoration: none;
    color: white;
}

/*=========FORGOT PASSWORD PAGE STYLING ========*/
#submitEmailBtn{
    background: none;
}
```

### Side Navbar push content when open
The navbar had been overlapping with the page content when opened. My task was to push the content when the navbar opens, while making the content responsive so that the user would not need to scroll horizontally. To do this, I used Javascript to add a class to the container when the navbar opens up. This allowed me to change the styling in a responsive manner.

```
site.css

/*===== MOVE BODY WHEN NAVBAR OPENS =====*/
#dashboardContainer .originalContainer {
    transform: translateX(0);
}

#dashboardContainer.moveContainer{
    transform:translateX(160px);
    width: calc(100% - 160px);
}
```

```
site.js

// BEGIN SIDE NAVBAR OPEN/CLOSE
$(document).ready(function () {
    $('#open-menu').on('click', function () {
        $('#menu').toggleClass('active');
        //alert("Menu open");
        $('#left-half').css({ "margin-left": "-95px" });
        $('.originalContainer').toggleClass('moveContainer');
    
    });
});

$(document).ready(function () {
    $('#dismiss').on('click', function () {
        $('#menu').toggleClass('active');
        //alert("Menu close");
        $('#left-half').css({ "margin-left": "-10px" });
        $('.originalContainer').toggleClass('moveContainer');
    });
});  
```


### Calendar styling issues
The app had a calendar that displayed employees' work schedules. I was tasked with fixing certain styling issues. Firstly, the calendar rendered with a vertical scrollbar. I changed the content height to auto so that it fits better within its container. The buttons at the top of the calendar were also aligned to the left. Ispaced it out so that it fits more evenly across the calendar. Lastly, the save event modal popped up to the side of the page and had to be dragged over to be seen by the user. I fixed this by changing the positioning of the modal. I also changed the width of its content to better fit the modal.

```
/*===== SPACE OUT THE BUTTONS AT THE TOP OF THE CALENDAR =====*/
.fc-header-toolbar {
        font-size: 14px;
        width: 100%; /*--expand toolbar to full width to spread buttons out evenly--*/
 }
    
.fc button { /*calendar toolbar button styling*/
        background: var(--dark-blue);
        color: white;
        font-weight: 500;
        font-family: "Trebuchet MS", Helvetica, sans-serif;
        text-shadow: none;
        box-shadow: none;
 }
 
 /*===== CENTER SAVE EVENT MODAL ===*/
 #formModal {
        width: 100%;
 }

 #modalPositioning {
        left: 50px;
        top: 5px;
 }

/*======================= RESPONSIVE CALENDAR HEADER ========================*/
    @media screen and (max-width: 1054px) {
        .fc-toolbar .fc-center {position: absolute; left: 35%; top: 7%;}
        .fc button {height: auto; padding: 0 .3em;}
        #calendar {padding-top: 80px;}
    }

    @media screen and (max-width: 818px) {
        .fc-left {padding-bottom: 10px; margin-left: 39%;}
        #calendar .fc-toolbar .fc-right {float: left;}
        .fc-toolbar .fc-center {left: 30%;}
        .fc-right {margin-left: 8%;}
    }
   

```

```
index.cshtml

function GenerateCalendar(events) {
                $('#calendar').fullCalendar({
                    contentHeight: "auto", // TO REMOVE CALENDAR VERTICAL SCROLLBAR
                    )},
                }
```

