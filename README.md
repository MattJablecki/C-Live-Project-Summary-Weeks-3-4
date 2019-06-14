# C-Sharp-Live-Project-Summary-Weeks-3-4

## Introduction
For my second two-week sprint on the Erectors Construction Project, I was assigned specifically to Front-End Development. After a conversation with my project manager, I was tasked with creating a drag and drop page in which an administrator could build a weekly schedule for all employees by simply setting the date range and dragging the employee into a bin containing their job for that week. Upon successful completion, I was then given an opportunity to do a similar page for the admins to assign a foreman to each job and set a date range for the entire life expectancy of the job they were assigned to. This was a fun and exciting challenge, as I had never had any exposure to drag and drop functionality. While I only worked on the front end development of this assignment, I will be spending the next two weeks implementing the back end functionality to make these pages operate correctly. 

Below you'll find a brief description of each page, as well as the code I used to create them.


## Schedule Creation Pages
* [Create Weekly Schedule](#create-weekly-schedule)
* [Assign Foreman](#assign-foreman)



### Create Weekly Schedule
For this page, the idea behind its necessity was simple. As it was, there was a page in which all the information for schedule building needed to be input manually, whether through drop down list selection or typing the information by hand. Also, only one schedule could be created at a time, which would be incredibly time consuming for a large business with many employees. With the new drag and drop page, the schedule creator can now set the work week at the top of the page, drag the employees from a database generated list of employees, and drop them into the job they will be assigned to. The jobs are also generated from the database, allowing for the admin to add both employees and jobs to the database and still generate an up to date list of both employees and jobs. In order to generate these two models into a single view, I needed to first get the lists of employees and jobs. This was accomplished using the following code:

        public List<ApplicationUser> GetEmployees()
        {
            List<ApplicationUser> employees = new List<ApplicationUser>();
            employees = db.Users.ToList();
            return employees;
        }

        public List<Job> GetJobs()
        {
            List<Job> jobs = new List<Job>();
            jobs = db.Jobs.ToList();
            return jobs;
        }

Once the lists were created, I needed to find a way to pass both lists to a single view, which was a new challenge for me. After a little research, I discovered Expando Objects. Among other things, Expando Objects allow you to create dynamic models that utilize properties from individual models in the same view. The code for generating this dynamic model and passing it to the view looks like this:


        public ActionResult MasterScheduleEdit()
        {
            dynamic myModel = new ExpandoObject();
            myModel.Employees = GetEmployees();
            myModel.Jobs = GetJobs();
            var weeks = FetchWeeks(2019);
            ViewBag.Weeks = new SelectList(weeks);
            return View(myModel);
        }


You'll notice that in the above code block, I also call a method, called FetchWeeks, and assign the value to a new SelectList using ViewBag. This created a list of weeks, Monday-Sunday, for the admin to choose from (using a dropdown) and set the date range for that week's schedule. The method itself is written below:

        public List<string> FetchWeeks(int year)
        {
            List<string> weeks = new List<string>();
            DateTime startDate = new DateTime(year, 1, 1);
            startDate = startDate.AddDays(1 - (int)startDate.DayOfWeek);
            DateTime endDate = startDate.AddDays(6);
            while (startDate.Year < 1 + year)
            {
                weeks.Add(string.Format("{0:MM/dd/yyyy} to {1:MM/dd/yyyy}", startDate, endDate));
                startDate = startDate.AddDays(7);
                endDate = endDate.AddDays(7);
            }
            return weeks;
        }

Finally, with all of the controller code written, it was time to create the view:


    @using ConstructionNew.Models;
    @model dynamic
    @{
        ViewBag.Title = "Create Weekly Schedule";
    }
    <link href="~/Content/Site.css" rel="stylesheet" />
    <h1 id="assignHeader">Create Weekly Schedule</h1>
        <div id="weekDDL">
            @Html.DropDownList("weeks", new SelectList(ViewBag.Weeks).ToString())
        </div>
        <div class="wrapper">
            <!-- Sidebar -->
            <div class="row">
                <div id="sidebar2">
                    <div class="sidebar-header">
                        <h3>Employees</h3>
                    </div>
                    <ul id="employeesMaster" class="sortable_list connectedSortable">
                        @foreach (ApplicationUser user in Model.Employees)
                        {
                            <li>@Html.DisplayFor(modelItem => user.UserName)</li>
                        }
                    </ul>
                </div>
                <div class="container" id="jobBox">
                    <div class="jobBoxheader"><h3>Jobs</h3></div>
                    @foreach (Job job in Model.Jobs)
                    {
                        <div id="sidebar">
                            <div class="sidebar-header">
                                <h3>@Html.DisplayFor(modelItem => job.JobTitle)</h3>
                                <h5>Job #: @Html.DisplayFor(modelItem => job.JobNumber)</h5>
                            </div>
                            <ul class="sortable_list connectedSortable"><li id="no">Drag User Here</li></ul>
                        </div>
                    }
                </div>
            </div>
        </div>
        <input type="submit" value="Save Changes" onclick="return confirm('Save Changes?')" />
       
Using razor, I utilize the dynamic model to pass the values for Users(employees) and Jobs. Then inside foreach loops, it's just a matter of specifying which model from the dynamic model is being referenced. Once the page was properly populated, the last step was implementing the JavaScript necessary to make the employees draggable. JQuery UI allows you to connect lists by envoking the .sortable property. This allowed me to pass employees to the different jobs, as well as in between the different jobs and back again. The script looks like this: 

    <script src="~/Scripts/jquery-3.3.1.js"></script>
    <script src="~/Scripts/jquery-ui.js"></script>
    <script>
        $(document).ready(function () {
            $(".sortable_list").sortable({ connectWith: ".connectedSortable", cancel: "#no" });
        });
    </script>

Lastly, I added some CSS to make the list items grow and change colors upon hover, to clearly differentiate between the selected list item and the others. The finished product looks like this:

![CreateWeek](https://user-images.githubusercontent.com/46763228/58728377-9edd8d00-839b-11e9-92ba-41f8e1e658c5.png)
![weekdrag](https://user-images.githubusercontent.com/46763228/58728459-cc2a3b00-839b-11e9-89a4-6e501ad9a2c3.png)
![createdrop](https://user-images.githubusercontent.com/46763228/58728621-2925f100-839c-11e9-8c0a-79934477045c.png)

 ### Assign Foreman
With the Create Weekly Schedule page up and running, I was then tasked with creating a similar page, only this time for assigning a Foreman to each job for the entire duration of a job. The skeleton of the page was quite similar, with a couple of minor tweaks to make the page function as needed. One of these tweaks was eliminating the drop down for weeks and replacing it with a date picker for each individual job. This was added by using a simple input type of "date" in a form, like so:

        <div class="sidebar-header">
            <h3>@Html.DisplayFor(modelItem => job.JobTitle)</h3>
            <h5>Job #: @Html.DisplayFor(modelItem => job.JobNumber)</h5>
            <form>
                Start Date: <input type="date" value="startDate" style="margin-bottom:8px;" />
            </form>
            <form>
                End Date: <input type="date" value="endDate" style="margin-left: 4px; margin-bottom:8px;" />
            </form>
        </div>

A minor change was needed in the script, as well, because only one Foreman needs to be assigned per job, as opposed to multiple employees in the Create Weekly Schedule page. 
        
    $(document).ready(function () {
        $(".sortable_list").sortable({
            connectWith: ".connectedSortable",
            cancel: "#no",
            receive: function (event, ui) {
                if ($(this).children().length > 2) {
                    $(ui.sender).sortable('cancel');
                }
            }
        }).disableSelection();
    });
    
By adding that receive function, the list can only have two items. One being the dummy item which directs the user where the drop zone is, and the second item being the Foreman you drop into the list. Additionally, I added logic to ensure that only Foreman would populate into the Employee list.

                <ul id="employeesMaster" class="sortable_list connectedSortable">
                    @foreach (ApplicationUser user in Model.Employees)
                    {
                        if (user.WorkType.ToString() == "Foreman")
                        {
                            <li>@Html.DisplayFor(modelItem => user.UserName)</li>
                        }
                    }
                </ul>
                
I also employed logic to determine whether a job was actually active by pulling the job's number. There are assignments such as PTO, Injured, Laid-Off, which do not require a Foreman for obvious reasons. To make this determination, the code looks for a job number. If one does not exist, the assignment does not populate.

    @foreach (Job job in Model.Jobs)
                {
                    if (job.JobNumber != null)
                    {
                    <div id="sidebar">
                        <div class="sidebar-header">
                            <h3>@Html.DisplayFor(modelItem => job.JobTitle)</h3>
                            <h5>Job #: @Html.DisplayFor(modelItem => job.JobNumber)</h5>
                            <form>
                                Start Date: <input type="date" value="startDate" style="margin-bottom:8px;" />
                            </form>
                            <form>
                                End Date: <input type="date" value="endDate" style="margin-left: 4px; margin-bottom:8px;" />
                            </form>
                        </div>
                        <ul class="sortable_list connectedSortable"><li id="no">Drag Foreman Here</li></ul>
                        <button type="button" style="margin-bottom:8px;" onclick="return confirm('Save Changes?')">Save Changes</button>
                    </div>
                    }
                 }
                 
The final result looks very simmilar to the Create Weekly page, with the aforementioned minor adjustments.

![foreman](https://user-images.githubusercontent.com/46763228/58728622-2925f100-839c-11e9-859b-c60bc22609bb.png)
![foremandrag](https://user-images.githubusercontent.com/46763228/58728623-2925f100-839c-11e9-85d2-db0b41b3ecaa.png)



        
### Summary
This sprint was highly enjoyable and tested my Front-End mettle. Design, by nature, is highly subjective. Therefore, determining what is "good" can be interpreted by different people in many different ways. My goal was to create something clean and user-friendly, with a touch of style that wasn't overpowering, flashy, or counter-productive to the project's overall aesthetic. Hopefully the client is happy with what I produced, and I look forward to making all the Back-End functionality in the upcoming weeks. 
