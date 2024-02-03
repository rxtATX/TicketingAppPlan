* *Note:* This is a living document. Please open Issues on this repository to provide feedback.

## Table of Contents 

- [Initialization](#initialization)
  * [Structure](#structure)
  * [Utilities](#utilities)
- [Models](#models)
  * [User](#user)
  * [User structure:](#user-structure-)
  * [Ticket](#ticket)
  * [Ticket structure:](#ticket-structure-)
  * [Log](#log)
  * [Log structure:](#log-structure-)
  * [Index](#index)
    + [User to Ticket](#user-to-ticket)
    + [Log to User](#log-to-user)
    + [Log to Ticket](#log-to-ticket)
  * [Seeds](#seeds)
- [Routing](#routing)
  * [Handlebars](#handlebars)
    + [Login](#login)
    + [Dashboard](#dashboard)
    + [Individual Ticket Page](#individual-ticket-page)
  * [Data](#data)
    + [Users](#users)
    + [Tickets](#tickets)
    + [Logs](#logs)
    + [Index.js](#indexjs)
  * [Controllers](#controllers)
    + [User Controllers](#user-controllers)
    + [Ticket Controllers](#ticket-controllers)
    + [Log Controllers](#log-controllers)
  * [Helpers](#helpers)
    + [Format Date](#format-date)
    + [Format Timestamp](#format-timestamp)
    + [Determine Class](#determine-class)
    + [Find Diff](#find-diff)
    + [With Auth](#with-auth)
  * [Views](#views)
    + [Pages](#pages)
      - [Login](#login-1)
      - [Home](#home)
      - [Individual Tickets](#individual-tickets)
    + [Layouts](#layouts)
      - [Login](#login-2)
      - [Main](#main)
    + [Partials](#partials)
  * [Public](#public)
    + [Scripts](#scripts)
      - [login.js](#loginjs)
      - [home.js](#homejs)
      - [ticket.js](#ticketjs)
- [Tests](#tests)
- [Deployment](#deployment)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

# Initialization

In this section, we establish the major structure for the application. 

These features will be needed by any and all developers collaborating on this project as foundations before features will be implemented.

- [ ] Initialize application with `npm init`, then install dependencies.
    - express
    - express-handlebars
    - sequelize
    - dotenv
    - connect-session-sequelize
    - bcrypt
    - mysql2
    - express-session
    - nodemon (dev-only)
- [ ] Add a .gitignore for Node.
- [ ] Create initial folder structure:
## Structure
```
├── controllers
│   ├── logControllers.js
│   ├── ticketControllers.js
│   ├── userControllers.js
├── routes
│   ├── index.js
│   ├── handlebarsRoutes.js
│   ├── apiRoutes
│   │   ├── index.js
│   │   ├── logRoutes.js
│   │   ├── ticketRoutes.js
│   │   ├── userRoutes.js
├── views
|   ├── home.handlebars
|   ├── login.handlebars
|   ├── ticket.handlebars
|   ├── layouts
│   │   ├── main.handlebars
│   │   ├── login.handlebars
|   ├── partials
│   │   ├── drawer.handlebars
│   │   ├── form.handlebars
│   │   ├── table.handlebars
│   │   ├── timeline.handlebars
├── models
|   ├── schema.sql
|   ├── index.js
|   ├── User.js
|   ├── Ticket.js
|   ├── Log.js
├── public
│   ├── assets
│   │   ├── css
│   │   │   ├── styles.css
│   │   ├── js
│   │   │   ├── login.js
│   │   │   ├── home.js
│   │   │   ├── ticket.js
├── node_modules
├── utils
│   ├── connection.js
│   ├── seeds.js
│   ├── helpers.js
├── .env
├── server.js
├── package.json
├── package-lock.json 
└── .gitignore
```
## Utilities
- [ ] Create a schema for the Sequelize models in `models/schema.sql` called `ticketing_db`.
- [ ] Set up a basic express server: `server.js`, using Express boilerplate basics.
- [ ] Add the express-handlebars tool as the view engine to server.js.
- [ ] In the `utils` folder, create a `connection.js`.
    - This file needs to connect to our MySQL database and export the connection.
- [ ] Modify the server.js file to complete the connection to our MySQL database before listening for client requests.
- [ ] Modify the server.js file to use express-session to allow for user session data within the routes/controllers.
- [ ] In the `.env` file, define `DB`, `USER`, and `PW` variables referenced within `utils/connection.js`.
- [ ] For convinience, modify the `package.json` file to include useful scripts.
    - "start": "node server.js"
    - "watch": "nodemon server.js"
    - "seed": "node utils/seeds.js"

# Models

In this section, we build several models for our database.

These models will be seeded through a JavaScript command, and can be tested with any client GUI.

## User
This model will store our user data.
## User structure: 
```
User
├── email
│   ├── STRING
│   ├── Required
│   ├── Must be email
├── password
│   ├── STRING
│   ├── Required
│   ├── Cannot be `password`
│   ├── Minimum of 6 characters
├── firstName
│   ├── STRING
│   ├── Required
├── lastName
│   ├── STRING
│   ├── Required
├── role
│   ├── STRING
│   ├── Required
│   ├── Must be `tech` or `client`
│   ├── Default value of `client`
```
- [ ] Complete the structure and type definitions.
- [ ] This User model must include an instance method using `bcrypt` to verify a password is valid upon login.
- [ ] This User model must include a `beforeCreate` hook to hash a password upon entry into the database.
- [ ] The `beforeCreate` hook must set the email to lower case upon entry into the database.
- [ ] This User model must include a `beforeUpdate` hook to hash a password upon changes to the database if the password is passed.
- [ ] The `beforeUpdate` hook must set the email to lower case upon changes to the database if the email is passed.

## Ticket
This model will hold our ticket data. Each ticket will be associated with a user (client) and another user (tech).
## Ticket structure:
```
Ticket
├── clientId
│   ├── INTEGER
│   ├── Required
│   ├── Foreign key which references `user`.`id`
├── techId
│   ├── INTEGER
│   ├── Foreign key which references `user`.`id`
├── subject
│   ├── STRING
│   ├── Required
├── description
│   ├── STRING
│   ├── Required
├── status
│   ├── STRING
│   ├── Required
│   ├── Must be `Open`, `Pending`, `Resolved`, or `Claimed`
│   ├── Default value of `Open`
├── urgency
│   ├── STRING
│   ├── Required
│   ├── Must be `Low`, `Medium`, or `High`
│   ├── Default value of `Low`
├── isArchived
│   ├── BOOLEAN
│   ├── Required
│   ├── Default value of `false`
```
- [ ] Complete the structure and type definitions.
- [ ] This Ticket model must include an `afterCreate` hook to update the `Log` model to reflect a newly created ticket. Params include `data`.
  * The `Log`.`type` will be "Created".
  * The `Log`.`message` will be "Ticket number `ticketId` created.
  * The `Log`.`userId` field's value will be passed as the `clientId` property on the argument to this method.
  * The `Log`.`ticketId` field will be a reference to the `id` property on the argument to this method.
- [ ] This Ticket model must include an `afterUpdate` hook, which will check if he ticket `status` has been changed to 'Resolved' and additionally set the `isArchived` proptery to true. 
- [ ] This Ticket model must include a `logChange` instance method to update the `Log` model to reflect a change to the ticket. Params include `userId` and `oldData`.
  * This method will use the `findDiff` helper method, passing along the changed ticket instance and the previous ticket values.
  * This method will return early if no changes are found.
  * The `Log`.`type` will be "Modified"
  * The `Log`.`message` will be "`findDiff.length` changes were made on `date` by `user`. `iterate the results of findDiff here`"
  * The `Log`.`userId` field's value will be passed as the argument to this method
  * The `Log`.`ticketId` field will be a reference to the instance `id`

## Log
This model will contain our log data. Each log will be associated with a ticket as well as a user.
## Log structure:
```
Log
├── userId
│   ├── INTEGER
│   ├── Required
│   ├── Foreign key which references `user`.`id`
├── ticketId
│   ├── INTEGER
│   ├── Required
│   ├── Foreign key which references `ticket`.`id`
├── message
│   ├── STRING
│   ├── Required
├── type
│   ├── STRING
│   ├── Required
│   ├── Default value will be "Message"
│   ├── Must be "Created", "Modified", or "Message"
├── isHidden
│   ├── BOOLEAN
│   ├── Required
│   ├── Default value of `false`
```
- [ ] Complete the structure and type definitions.

## Index
The `models/index.js` file will define and export the relationships between these models.

### User to Ticket
- User has many references to Ticket
- Ticket belongs to a User
- Two foreign key definitions
    1. 'clientId' as 'client'
    2. 'techId` as 'tech'
- [ ] Complete
### Log to User
- User has many Log
- Log belongs to a User
- Foreign key definition of 'userId'
- [ ] Complete
### Log to Ticket
- Ticket has many Log
- Log belongs to Ticket
- Foreign key definition of 'ticketId'
- [ ] Complete

## Seeds
All users of this application will be seeded directly by a DBA - we are emulating a business domain where the general population is not able to enroll with our application through signing up.

- [ ] Seed sample user data. Ensure that all hooks run upon creation.
- [ ] (optional) Seed sample ticket data. Ensure all hooks run upon creation.
- [ ] (optional, conditional on if ticket seeds exist) Seed sample log data.

# Routing
## Handlebars
The current plan includes three unique views. These routes will all be of type GET, and will perform the necessary MySQL queries (Sequelize) before rendering.

Each of these views will receive the required values based on context, but also
1. loggedIn: BOOLEAN
2. title: STRING
3. layout: STRING
4. userType: STRING

### Login
- [ ] The Login view will be displayed when the URL path is '/login'.
- [ ] This view will be rendered with the login view, on the login layout, with the title of 'Log In' and no user type defined.
- [ ] If the user is already logged in, they will be automatically redirected away from this view to the Dashboard page instead.
- [ ] No Sequelize queries will need to run in order to render this view, however this view will be connected to the login.js script, which will post a request to authenticate using Sequelize to our server upon form submission.
- [ ] No userType will be added to this particular view.

### Dashboard
- [ ] The Dashboard view will be displayed when the URL path is '/:status?'.
- [ ] This view will be rendered with the home view, the main layout, the title of 'Dashboard', and whichever user type the user authenticated with.
- [ ] If the user is not logged in, they will be automatically redirected away from this view to the Login page instead though the `withAuth` middleware.

If the user is signed in as a `client`:
  * - [ ] We will need to query to the Ticket model where `clientId` matches the `user`.`userId` of the signed in user from the session object, and include User data.
  * - [ ] Only tickets where their user id is present on the 'clientId' field will be included.

If the user is signed in as a `tech`:
  * - [ ] We will need to query to the Ticket model and include User data.
  * - [ ] Tickets where their user id is present on the 'techId' field will be included, as well as any tickets which have the 'status' of 'Open'.

- [ ] If the status parameter is applied to the request, filter the tickets to only those whose status value match the request.
- [ ] All tickets should include `client` and `tech` `firstName` `lastName` `id` and `role` from associated Users.
- [ ] Only tickets which have not been archived should be included in these results.
- [ ] We will need to serialize the data before the view renders.

### Individual Ticket Page
- [ ] The Individual Ticket view will be displayed when the URL path is '/ticket/:id'.
- [ ] This view will be rendered with the ticket view, the main layout, the title of 'Ticket Details', and whichever user type the user authenticated with.
- [ ] If the user is not logged in, they will be automatically redirected away from this view to the Login page instead through the `withAuth` middleware.
- [ ] If the user is signed in as a `client`, they will not be allowed to view unassociated tickets; if the ticket's `clientId` property doesn't match their `id`, they will be automatically redirected back to the `home` view.
- [ ] All tickets should include `client` and `tech` `firstName` `lastName` `id` and `role` from associated Users and all `Log` data for this ticket.
- [ ] If the ticket has been archived, redirect the user back to `home` view
- [ ] We will need to serialize the data before the view renders.

## Data
Our API routing will only include routing behavior, splitting our paths and query methods. Then it will pass the actual Sequelize interaction to our controller files.

### Users
The routes will match '/api/user' to handle POST, and DELETE requests.

- [ ] POST will call the `loginUser` controller.
- [ ] DELETE will call the `logoutUser` controller.

In the future, we may implement PUT calls to allow users to change their passwords; not in current scope.
### Tickets
The routes will match '/api/ticket' to handle POST requests.

- [ ] POST will call the `createTicket` controller.

The routes will also match '/api/ticket/:id' to handle PUT, and DELETE requests.

- [ ] PUT will call the `editTicket` controller.
- [ ] DELETE will call the `archiveTicket` controller.

### Logs
The routes will match '/api/log/:ticketId?drawer=`BOOLEAN`' to handle POST calls.

- [ ] POST will call the `createLog` controller.

The routes will also match '/api/log/:ticketId/:logId?drawer=`BOOLEAN`' to handle PUT, and DELETE calls. 

- [ ] PUT will call the `editLog` controller.
- [ ] DELETE will call the `deleteLog` controller.

### Index.js
- [ ] This file will combine and export all API router behavior for easy attachment to the server.
- [ ] Ensure the express application applies these routes as middleware.

## Controllers
This section of our application will dispatch the appropriate Sequelize queries, then supply the server response to the client.

### User Controllers
#### loginUser 
- [ ] Perform a Sequelize `findOne` query on the `User` model.
- [ ] Pass along the client's email property as the argument. 
- [ ] Once data has been retrieved it will be confirmed that the supplied password matches the one stored in the database. 
- [ ] When a user successfully logs in, they will be redirected to the home page. 
- [ ] Their session data will be added to the express-session object. 

#### logoutUser 
- [ ] Destroy the express-session object.
- [ ] Redirect the client to the login page.

### Ticket Controllers
#### createTicket
- [ ] Perform the Sequelize `create` query on the `Ticket` model.
- [ ] Pass along the form data from the client as the argument. 
- [ ] Once data has been updated, the server will respond with a page redirection to the newly created ticket page.

#### editTicket 
- [ ] Perform the Sequelize `findByPk` query on the `Ticket` model.
- [ ] Pass along the `id` from the parameters list.
- [ ] `Set` and `Save` the changes from the request body. 
- [ ] If one of the changes includes adding a new `techId` to the ticket, the function should also automatically modify the `status` to `Claimed`.
- [ ] Invoke the logChange instance method with the `req.session.user` object and the ticket data from pre-`Set`.
- [ ] Once data has been passed to the database,the server will instruct the client to redirect back to the referrer location.

#### archiveTicket
- [ ] Perform the `findOne` query on the `Ticket` model
- [ ] Pass along the `id` from the parameters list, and the `status` value of `Resolved`. 
- [ ] `Set` and `Save` the changes from the request body. 
- [ ] Once the data has been updated, the server will instruct the client to redirect back to the referrer location.

### Log Controllers
For these log controllers, a query parameter ?drawer=true or ?drawer=false will indicate whether the chat log window should be open upon page refresh.

NOTE: Alternative to page refreshing is regular DOM manipulation. @@TODO: Research best practice in this scenario.

#### createLog

- [ ] Perform the `create` method on the `Log` model.
- [ ] Pass along the form data from the client as the request body. 
- [ ] The `userId` property should automatically be assigned to the currently authenticated user's `id` from the `req.session.user` object.
- [ ] The `ticketId` property value will come from the request parameters, `ticketId`.
- [ ] Once data has been updated, the server will respond with a page refresh on the individual ticket page.

#### editLog
- [ ] Perform the `findByPk` method on the `Log` model.
- [ ] Pass along the `logId` from the parameters list.
- [ ] `Set` and `Save` the changes from the request body. 
- [ ] Once the data has been updated, the server will respond with a page refresh on the individual ticket page.

#### deleteLog
- [ ] Perform the `destroy` method on the `Log` model.
- [ ] Pass along the `Ticket`.`id` from the parameters list. 
- [ ] Once the data has been removed from the database, the server will respond with a page refresh on the individual ticket page.

## Helpers
These custom helpers are tools which `express-handlebars` will be able to apply to our template views.

- [ ] We will need to modify the `server.js` file to apply these helpers to our handlebars view engine.

### Format Date
- [ ] The `formatDate` function should take in a `date` argument and return the date string in `Mo DD, YYYY hh:mm A` format.

### Format Timestamp
- [ ] The `formatTimestamp` function should take in a `date` argument and return the date string in `hh:mm A` format.

### Determine Class
- [ ] The `determineClass` function should take in a `value` argument and return the string representing the appropriate class determined by a switch case. This will be a general use function, such as adding the "hidden" or "shown" class to the hide/show eye icons in messages, or "left-align" or "right-align" on the message bubbles.

### Find Diff
- [] The `findDiff` function should take in a `newValue` and `oldValue` object argument. It will need to check the deep equivalence of these two obects and return an array of strings. 
- [ ] Each string should be constructed of "`keyname` was changed from `oldValue` to `newValue` by `activeUser`."

### With Auth
- [ ] The `withAuth` helper function will work as a route-specific middleware function.
- [ ] It should allow users who are currently logged in on the `req.session` object to proceed to the next route handler.
- [ ] It should redirect users who are not currently logged in back to the `login` view.

## Views
The views of the app will be controlled by Handlebars. We'll use the `express-handlebars` npm tool to assist. We have 3 primary views in the initial proposal, allowing authentiation, viewing logged in details, and a view to see the details of a ticket.

### Layouts
#### Login
- [ ] The Login view will have a standard HTML basic skeleton. 
- [ ] It will link to `style.css` and `login.js`. 
- [ ] Its title will be hard-coded to "Login". 
- [ ] It will render the view's body content as a variable.

#### Main
- [ ] The Main view will have a standard HTML basic skeleton. 
- [ ] It will link to `style.css`, `home.js`, and `ticket.js`. 
- [ ] Its title will be a variable reference to the title passed by the controller. 
- [ ] It will render the view's body content as a variable.

### Pages
#### Login
- [ ] Users should be presented with an authentication form.
    - [ ] The form should accept an email input.
    - [ ] The form should accept an password input.
    - [ ] The form should have a submit button which will cause the `login` function to fire.
![Login](./public/assets/wireframes/Login-View.PNG)
Login view.
- [ ] The layout should match the wireframe.
#### Home
- [ ] Users should be presented with a dashboard page.
- [ ] The top portion of the page is dedicated to several buttons. On click, these buttons will filter the table view to only include tickets which match the selected status. They will function as links which take the user to `/:status?`
    * All - takes the user to `/`
    * Open
    * Pending
    * Resolved
    * Claimed
- [ ] The bottom portion of the page will have a table showing ticket data depending on the current user type.
- [ ] If the user is a `client`, they will see only their own tickets in this table view.
- [ ] If the user is a `tech`, they will see any tickets which have the status of `open` or their own tickets.
- [ ] If the user is a `tech` and the ticket has a status of `Open`, they will see an additional column with a button they can click to claim.
- [ ] The table columns will include `Status`, `Urgency`, `Client`, `Technician`, `Date Created`.
- [ ] Each row on this table will be clickable, and will function as a link to take the user to the individual ticket page.
- [ ] When there is no ticket data to display as a technician, the table will show "No pending tickets. Check back soon."
- [ ] When there is no ticket data to display as a client, the table will show the same `create new ticket` form which also displays in the drawer.
- [ ] There will be a button in the center of the page with a plus sign on it, `addNewBtn`.
- [ ] When the center button is pressed, the side drawer will open. This side drawer renders the `create new ticket` form.
    * A Subject field
    * A Description multi-line input
    * An urgency dropdown - including Low, Medium, and High
    * A submit button
- [ ] When the `create new ticket` form is submitted, an API call will be made to `/api/ticket`. On complete, the user will be redirected to the individual ticket page.
- [ ] The drawer should have an "X" span, when clicked it will close the drawer element.
- [ ] The bottom of the page should have a "Logout" button, when clicked it will make an API call to `/api/user` to log the user out and to take the user back to the login page.
[Home](./public/assets/wireframes/Home-View.PNG)
Home page view. Drawer closed.
![Home with drawer](./public/assets/wireframes/Home-Drawer-View.PNG)
Home page view. Drawer open.
- [ ] The view should match the wireframe.
#### Individual Tickets
- [ ] Users should be presented with the individual ticket page.
- [ ] The top of the page features a header detailing the current ticket information.
    * The Subject
    * The Description
    * A badge representing the Urgency
    * A statement "Opened by" with the name of the client "on" with the Created At date
- [ ] The bottom left of the page should display the `update ticket form`. This form will come pre-populated with the current ticket data.
    * A Subject input
    * A Description multi-line input
    * An Urgency dropdown will include Low, Medium, and High
    * A Status dropdown will include Open, Claimed, Pending, and Resolved
        * A client is only able to select "Resolved", closing their ticket and indicating their satisfaction with the result.
        * If the Status is changed to Open by a tech, any tech data attached to the ticket will be removed.
        * If the status is changed to Claimed by a tech, their tech information will be added to the ticket
        * If the status is changed to Pending, no additional changes will be made to the ticket - this is simply a status where in future iterations we may send email or other alerts to the client to inform them of the technician's belief that the ticket is resolved for confirmation
        * The technician is not able to change the ticket to Resolved.
    * A Save Changes button
- [ ] When the `update ticket form` submits, an API call will be made to `/api/ticket/:id`
- [ ] The bottom right of the view will display a vertical timeline element. This entity will be a visual display of the log history of the ticket.
- [ ] Each timeline item will include the `Log`.`type` field in the element main body. Under in subtext, it will also include the `Created At` timestamp for the log.
- [ ] When the user clicks on any timeline element, the side drawer will open.
- [ ] The side drawer will display the log history in long-form.
- [ ] If the `Log`.`type` is "Message", depending on if the current user sent the message, the chat bubble will be oriented in one direction or another.
    * If the current user sent the message, it will be oriented to the right.
    * If the current user sent the message, it will be oriented to the left.
- [ ] If the `Log`.`type` is "Created" or "Modified", the message will be center aligned and will say "This ticket was `type`ed on `Created At`."
- [ ] If the user sent the message, the bubble will also have a "show/hide" toggle icon. When shown, a message will also appear on the other user's chat log. When hidden, it will not be shown to anyone except the creator of the message. When clicked, an API call will be made to `/api/log/:ticketId/:logId`
- [ ] Each message will have a timestamp.
- [ ] The drawer will have a form element at the bottom.
    * A multi-line input
    * A "Hide" checkbox icon
    * A submit button, when clicked an API call will be made to `/api/log/:ticketId`
- [ ] The drawer should have an "X" span, when clicked it will close the drawer element.
- [ ] The bottom of the page should have a "Back" button, when clicked it will function as a link to take the user back to the home page.
- [ ] The bottom of the page should have a "Logout" button, when clicked it will make an API call to `/api/user` to log the user out and to take the user back to the login page.
![Login](./public/assets/wireframes/Ticket-View.PNG)
![Login with drawer](./public/assets/wireframes/Ticket-Drawer-View.PNG)
- [ ] The view should match the wireframe.

### Partials
Partials may be used in this application for re-useable elements referenced in the [Views](#views) section.

## Public
The `public` folder will hold the static assets, including the wireframe images mocking up this application, the stylesheets, and the front-end javascript.

### Scripts
#### login.js
This JS file will handle the login form submit behavior.

- [ ] Listen for submit events on the `loginForm` element. When the submit event occurs, capture the form field data and POST to `/api/user`

#### home.js
This JS file will handle the `logout`, `open drawer`, `create new ticket`, and `claim ticket` behavior.

- [ ] Listen for click events on the `openDrawer` element. When the click event occurs, open the side drawer element.
- [ ] Listen for submit events on the `createNewTicket` element. When the submit event occurs, capture the form field data and POST to `/api/ticket`
- [ ] Listen for click events on the `claimTicket` button elements. When the click event occurs, capture the ticket id data and PUT to `/api/ticket/:id`
- [ ] Listen for click events on the `logout` button element. When the click event occurs, make a DELETE call to `/api/user`

#### ticket.js
This JS file will handle the `update ticket`, `add message`, and `toggleHideMessage` behavior.

- [ ] Listen for click events on the `openDrawer` element. When the click event occurs, open the side drawer element.
- [ ] Listen for submit events on the `updateTicket` element. When the submit event occurs, capture the form field data and PUT to `/api/ticket/:ticketId`
- [ ] Listen for submit events on the `addMessage` element. When the submit event occurs, capture the form field data and POST to `/api/log/:ticketId?drawer=<BOOLEAN>`
- [ ] Listen for click events on the `toggleHideMessage` button elements. When the click event occurs capture the log id and make a PUT call to `/api/log/:ticketId/:logId?drawer=<BOOLEAN>`
- [ ] Listen for click events on the `logout` button element. When the click event occurs, make a DELETE call to `/api/user`

### Tests

Consider adding tests for this application.

### Deployment

Provision a JAWSDB on Heroku.
Deploy to Heroku.