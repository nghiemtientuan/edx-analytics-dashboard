edX Analytics Dashboard [![BuildStatus](https://travis-ci.org/edx/edx-analytics-dashboard.svg?branch=master)](https://travis-ci.org/edx/edx-analytics-dashboard) [![CoverageStatus](https://img.shields.io/coveralls/edx/edx-analytics-dashboard.svg)](https://coveralls.io/r/edx/edx-analytics-dashboard?branch=master)
=======================
Dashboard to display course analytics to course teams

Prerequisites
-------------
* Python 3.5.x
* [gettext](http://www.gnu.org/software/gettext/)
  
        $ $ apt install gettext
  
* [node](https://nodejs.org) 12.11.1
* [npm](https://www.npmjs.org/) 6.11.3
* [JDK 7+](http://openjdk.java.net/)
  
        $ sudo apt-get install openjdk-8-jre (or openjdk-7-jre)
        $ apt install libpython3.5-dev

Warning: You must have NPM version 5.5.1. Using another version might result in
a different `package-lock.json` file.

        $ pip install virtualenv
        $ virtualenv venv
        $ virtualenv -p python3 venv
        $ source venv/bin/activate
        $ python3 -m pip install --upgrade pip==9.0.3

        $ pip install nodeenv
        $ nodeenv --node=12.11.1 --npm=6.11.3 -p

Getting Started
---------------
1. Get the code (e.g. clone the repository).
   
        $ git clone https://github.com/nghiemtientuan/edx-analytics-dashboard.git
        $ git checkout git checkout open-release/juniper.master
   
2. Install the Python/Node requirements:

        $ make develop

3. Setup your database:

        $ make migrate

4. Run the webpack-dev-server:

        $ npm start

5. Run resources server:
   
        $ DJANGO_DEV_SERVER='http://localhost:9000' npm start

6. (run in other terminal) In a separate terminal run the Django development server:

        $ ./manage.py runserver 0.0.0.0:9000

(optional) By default the Django Default Toolbar is disabled. To enable it set the environmental variable ENABLE_DJANGO_TOOLBAR. Alternatively, you can launch the server using:
                
        $ ENABLE_DJANGO_TOOLBAR=1 ./manage.py runserver 0.0.0.0:9000

Visit http://localhost:9000 in your browser and then login through the LMS to access Insights (see **Authentication & Authorization** below for more details).

Setup Authentication & Authorization
------------------------------
1. Setup Authentication & Authorization in edx-analytic-data-api
2. Setup application lms admin (/18000/admin) page:
    
    2.1 Add application in http://localhost:18000/admin/oauth2_provider/application
            
            {
                Client id: "clien_id_string_auto_random" (auto_random)
                user: 3
                redirect uri: http://localhost:9000/complete/edx-oauth2/
                type: confidential
                grant type: authorization code
                Client secret: "clien_secret_string_auto_random" (auto_random)
                name: analytic-dashboard-sso
                tick: Skip authorization
            }

    2.2 Add application access in http://localhost:18000/admin/oauth_dispatch/applicationaccess
            
            {
                Application: analytic-dashboard-sso
                Scopes: "user_id"
            }

3. Copy Client id and Client secret

    Edit file in analytics_dashboard/settings/base.py:

        SOCIAL_AUTH_EDX_OAUTH2_KEY = "clien_id_string_auto_random"
        SOCIAL_AUTH_EDX_OAUTH2_SECRET = "clien_secret_string_auto_random"

4. Using postman to create api(get) http://localhost:9000/api-token-auth to get token

    Edit file in analytics_dashboard/settings/base.py:
   
        DATA_API_AUTH_TOKEN = 'paste_token_here'

Setup in server
------------------------------

Edit file analytics_dashboard/settings/base.py and analytics_dashboard/settings/dev.py

Feature Gating
--------------
See the [Waffle documentation](http://waffle.readthedocs.org/en/latest/) for
details on utilizing features in code and templates.

The following switches are available:

| Switch                               | Purpose                                               |
|--------------------------------------|-------------------------------------------------------|
| show_engagement_forum_activity       | Show the forum activity on the course engagement page |
| enable_course_api                    | Retrieve course details from the course API           |
| enable_ccx_courses                   | Display CCX Courses in the course listing page.       |
| enable_engagement_videos_pages       | Enable engagement video pages.                        |
| enable_video_preview                 | Enable video preview.                                 |
| display_course_name_in_nav           | Display course name in navigation bar.                |
| enable_performance_learning_outcome  | Enable performance section with learning outcome breakdown (functionality based on tagging questions in Studio) | 
| enable_learner_download              | Display Download CSV button on Learner List page.     |
| enable_problem_response_download     | Enable downloadable CSV of problem responses          |
| enable_course_filters                | Enable filters (e.g. pacing type) on courses page.    |
| enable_course_passing                | Enable passing column on courses page.                |

Asset Pipeline
--------------
Static files are managed via [webpack](https://webpack.js.org/).

To run the webpack-dev-server, which will watch for changes to static files
(`.js`, `.css`, `.sass`, `.underscore`, etc. files) and incrementally recompile
webpack bundles and try to hot-reload them in your browser, run this in a
terminal:

    $ npm start

### Acceptance Tests
The tests make a few assumptions about URLs and authentication. These can be overridden by setting environment variables
when executing either of the commands above.

| Variable                     | Purpose                                    | Default Value                    |
|------------------------------|--------------------------------------------|----------------------------------|
| DASHBOARD_SERVER_URL         | URL where the dashboard is served          | http://127.0.0.1:9000            |
| API_SERVER_URL               | URL where the analytics API is served      | http://127.0.0.1:8000/api/v0     |
| API_AUTH_TOKEN               | Analytics API authentication token         | edx                              |
| DASHBOARD_FEEDBACK_EMAIL     | Feedback email in the footer               | override.this.email@example.com  |
| TEST_USERNAME                | Username used to login to the app          | edx                              |
| TEST_PASSWORD                | Password used to login to the app          | edx                              |
| PLATFORM_NAME                | Platform/organization name                 | edX                              |
| APPLICATION_NAME             | Name of this application                   | Insights                         |
| SUPPORT_EMAIL                | Email where error pages should link        | support@example.com              |
| ENABLE_COURSE_API            | Indicates if the course API is enabled on the server being tested. Also, determines if course performance tests should be run. | False     |
| GRADING_POLICY_API_URL       | URL where the grading policy API is served | (None)                           |
| COURSE_API_URL               | URL where the course API is served         | (None)                           |
| COURSE_API_KEY               | API key used to access the course API      | (None)                           |
| ENABLE_OAUTH_TESTS           | Test the OAUTH sign-in process             | true                             |
| ENABLE_AUTO_AUTH             | Sign-in using auto-auth. (no LMS involved) | false                            |
| ENABLE_COURSE_LIST_FILTERS   | Tests on filtering the course list         | false                            |
| ENABLE_COURSE_LIST_PASSING   | Tests on the passing learners column in the course list | false               |


#### Course Validation
In addition to the standard acceptance tests, there is also a script to validate all course pages and report their
HTTP status codes. Use the command below to execute this script.

        $ make course_validation

