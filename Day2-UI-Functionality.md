# LAB: Build an Angular Component from Scratch

## Overview

In the previous exercises, you created and customized pages within an Automate Custom UI.

Now we're going to take a closer look at one of the most important building blocks of an Angular application: the **Component**.

Rather than starting with a completed component, we'll build one from scratch and progressively add functionality to it.

By the end of this lab, you will have worked with:

- Angular Components
- HTML Templates
- Component Properties
- Data Binding
- Event Binding
- Component Methods
- Component Lifecycle
- Component Imports
- Angular Routing

---

## Understanding What We're Building

An Angular component combines application logic with a user interface.

At a high level:

```text
TypeScript Component
        ↓
 Application Logic
        ↕
   HTML Template
        ↓
   User Interface
```

The **TypeScript component** controls what the component knows and what it can do.

The **HTML template** controls what the user sees.

We'll start with a very simple component and build upon it throughout this lab.

---

# Part 1: Create the Component

## Step 1: Create the Component Folder

In your Custom UI source code, navigate to:

```text
libs/
```

Create a new folder named:

```text
dashboard
```

Your directory should now contain:

```text
libs/
└── dashboard/
```

---

## Step 2: Create the TypeScript Component

Inside the new `dashboard` folder, create a file named:

```text
customDashComponent.ts
```

Add the following code template and complete the code:

```typescript
// import the component library
import { Component } from '@angular/core';

// add the component decorator that identifies this file as a component
@Component({
    selector: 'hxp-dashboard',
    templateUrl: './customDashComponent.html'
})

// export the class in order to define the component's data and behavior
export class customDashComponent {

}
```

Save the file.

### What Did We Just Create?

Let's look at some of the important parts of this component.

The following decorator tells Angular that this class represents a component:

```typescript
@Component({
```

The `selector` provides an identifier for the component:

```typescript
selector: 'hxp-dashboard'
```

The `templateUrl` tells Angular where the HTML template for this component can be found:

```typescript
templateUrl: './customDashComponent.html'
```

Finally, the class itself is where we can define the component's **data and behavior**:

```typescript
export class customDashComponent {

}
```

As we progress through this lab, most of our functionality will be added inside this class.

---

# Part 2: Create the Component's User Interface

## Step 3: Create the HTML Template

Our component references an HTML file that doesn't exist yet.

Inside the same `dashboard` folder, create:

```text
customDashComponent.html
```

Add:

```html
<h1>Claims Dashboard</h1>

<p>My dashboard component is working!</p>
```

Save the file.

Your component now consists of two files:

```text
dashboard/
├── customDashComponent.ts
└── customDashComponent.html
```

Think of these files as having two different responsibilities:

### TypeScript

```text
What does my component know?

What can my component do?
```

### HTML

```text
What does my component show?
```

As our component becomes more functional, this separation becomes increasingly important.

---

# Part 3: Make Angular Load the Component

Our component now exists, but Angular doesn't yet know **when to display it**.

To make the component accessible within our application, we'll create a route.

---

## Step 4: Open the Application Routes

Navigate to:

```text
libs/workspace-hxp/app-shell/src/lib/
```

Open:

```text
experience-workspace-app-shell.routes.ts
```

This file defines routes within the Angular application.

---

## Step 5: Import the Component

At the top of the file, add:

```typescript
import { customDashComponent } from '../../../../dashboard/customDashComponent';
```

### Why Is This Necessary?

We created the component, but the routing file does not automatically know that it exists.

The `import` makes our component available to the routing configuration.

---

# Part 4: Create a Route

## Step 6: Add the Dashboard Route

Locate the `APP_ROUTES` array.

Add the following route:

```typescript
{
    path: 'dashboard',
    component: customDashComponent
}
```

This creates the following relationship:

```text
/dashboard
     ↓
Angular Router
     ↓
customDashComponent
     ↓
customDashComponent.html
```

When Angular receives a request for the `/dashboard` route, it will load our component.

---

# Part 5: Test the Component

## Step 7: Start the Application

Ensure that all edited files are saved.

Open a Terminal window at the root directory of your Custom UI and run:

```bash
npm start workspace-hxp
```

Once the application starts, navigate to:

```text
http://localhost:4200/#/dashboard
```

You should see:

> # Claims Dashboard
>
> My dashboard component is working!

---

## Checkpoint

You have now created an Angular component completely from scratch.

You connected:

```text
Route
  ↓
Component
  ↓
Template
  ↓
Browser
```

Before continuing, make sure you understand the responsibility of each part.

---

# Part 6: Give the Component Data

Our component can display HTML, but the content is currently hard-coded directly into the template.

Let's move some of that information into the component.

---

## Step 8: Create a Component Property

Open:

```text
customDashComponent.ts
```

Update the component class:

```typescript
export class customDashComponent {

    dashboardTitle = 'Insurance Claims Dashboard';

}
```

Now open:

```text
customDashComponent.html
```

Replace:

```html
<h1>Claims Dashboard</h1>
```

with:

```html
<h1>{{ dashboardTitle }}</h1>
```

Save both files.

Your browser should now display:

> # Insurance Claims Dashboard

---

## What Just Happened?

The following property exists in our TypeScript component:

```typescript
dashboardTitle = 'Insurance Claims Dashboard';
```

Our HTML accesses that property using:

```html
{{ dashboardTitle }}
```

This is called **interpolation**.

Conceptually:

```text
COMPONENT

dashboardTitle
      │
      ▼
{{ dashboardTitle }}

TEMPLATE
```

Instead of the HTML deciding what the title is, the **component now owns that data**.

---

# Part 7: Add More Component Data

## Step 9: Add Claim Information

Let's give our component additional information.

Update the component:

```typescript
export class customDashComponent {

    dashboardTitle = 'Insurance Claims Dashboard';

    openClaims = 128;
    approvedToday = 34;
    underReview = 57;
    highPriority = 12;

}
```

Now update the HTML:

```html
<h1>{{ dashboardTitle }}</h1>

<h2>Claim Overview</h2>

<p>Open Claims: {{ openClaims }}</p>
<p>Approved Today: {{ approvedToday }}</p>
<p>Under Review: {{ underReview }}</p>
<p>High Priority: {{ highPriority }}</p>
```

Save both files.

Your dashboard should now display the values stored by the component.

The flow looks like this:

```text
TypeScript Data
       ↓
     Angular
       ↓
      HTML
       ↓
     Browser
```

Our component now **knows information** that can be presented within the UI.

Next, we'll make it actually **do something**.

---

# Part 8: Give the Component Behavior

## Step 10: Create a Component Method

Open:

```text
customDashComponent.ts
```

Add the following method:

```typescript
showMessage(): void {
    console.log('Dashboard button clicked!');
}
```

Your component should now look similar to:

```typescript
export class customDashComponent {

    dashboardTitle = 'Insurance Claims Dashboard';

    openClaims = 128;
    approvedToday = 34;
    underReview = 57;
    highPriority = 12;

    showMessage(): void {
        console.log('Dashboard button clicked!');
    }

}
```

Now open:

```text
customDashComponent.html
```

Add:

```html
<button (click)="showMessage()">
    Test Dashboard
</button>
```

Save both files.

---

## Step 11: Test the Method

Open your browser's Developer Tools and select the **Console**.

Click:

**Test Dashboard**

You should see:

```text
Dashboard button clicked!
```

We've now connected an event within the HTML to functionality within our component.

```text
HTML
 │
 │ click
 ▼
Component Method
 │
 ▼
Application Logic
```

The following Angular syntax:

```html
(click)="showMessage()"
```

is an example of **event binding**.

When the user clicks the button, Angular calls the `showMessage()` method in our component.

---

# Part 9: Let Component Behavior Change the UI

Logging information to the browser console is useful for testing, but let's make our component change something the user can actually see.

---

## Step 12: Create a Status Property

Add the following property to your component:

```typescript
statusMessage = '';
```

Now change the `showMessage()` method:

```typescript
showMessage(): void {
    this.statusMessage = 'Dashboard functionality is working!';
}
```

Your component should now contain:

```typescript
export class customDashComponent {

    dashboardTitle = 'Insurance Claims Dashboard';

    openClaims = 128;
    approvedToday = 34;
    underReview = 57;
    highPriority = 12;

    statusMessage = '';

    showMessage(): void {
        this.statusMessage = 'Dashboard functionality is working!';
    }

}
```

---

## Step 13: Display the Status

Update the HTML:

```html
<h1>{{ dashboardTitle }}</h1>

<h2>Claim Overview</h2>

<p>Open Claims: {{ openClaims }}</p>
<p>Approved Today: {{ approvedToday }}</p>
<p>Under Review: {{ underReview }}</p>
<p>High Priority: {{ highPriority }}</p>

<button (click)="showMessage()">
    Test Dashboard
</button>

<p>{{ statusMessage }}</p>
```

Save the files and click the button again.

The browser should display:

```text
Dashboard functionality is working!
```

---

## What Is Happening?

We now have communication flowing in both directions.

```text
USER
 │
 ▼
Button Click
 │
 ▼
showMessage()
 │
 ▼
statusMessage changes
 │
 ▼
Angular updates HTML
 │
 ▼
USER SEES CHANGE
```

This is an important transition.

We're no longer simply creating a web page.

We're creating an **application interface that can respond to the user**.

---

# Part 10: Introduce the Component Lifecycle

Angular components have a lifecycle.

There are times when we want functionality to occur automatically as the component is created or initialized.

---

## Step 14: Add Initialization Logic

Update the component:

```typescript
export class customDashComponent {

    dashboardTitle = 'Insurance Claims Dashboard';

    openClaims = 128;
    approvedToday = 34;
    underReview = 57;
    highPriority = 12;

    statusMessage = '';

    constructor() {
    }

    ngOnInit() {
        console.log('Dashboard component loaded.');
    }

    showMessage(): void {
        this.statusMessage = 'Dashboard functionality is working!';
    }

}
```

Save the file and reload the dashboard.

Open the browser console.

You should see:

```text
Dashboard component loaded.
```

---

## Constructor vs. ngOnInit

For now, think of these as two points where initialization can occur.

### `constructor()`

The constructor runs when the component is created.

```typescript
constructor() {

}
```

### `ngOnInit()`

`ngOnInit()` provides a place for initialization logic when the component starts.

```typescript
ngOnInit() {

}
```

We'll encounter these again as we add more functionality to our Custom UI.

---

# Part 11: Add Component Imports

Components frequently rely on functionality provided by Angular, Angular Material, ADF, or other components.

To use that functionality, we need to make it available to our component.

---

## Step 15: Add Imports

At the top of:

```text
customDashComponent.ts
```

add:

```typescript
import { Component } from '@angular/core';
import { MatDividerModule } from '@angular/material/divider';
import { HeaderComponent } from '@hxp/shared-hxp/navigation/header';
import { RouterLink } from '@angular/router';
```

> **NOTE:** If `Component` is already imported, do not add a second `Component` import. Replace the existing import section with the code above.

Now update the `@Component` configuration:

```typescript
@Component({
    selector: 'hxp-dashboard',
    templateUrl: './customDashComponent.html',
    imports: [
        MatDividerModule,
        HeaderComponent,
        RouterLink
    ]
})
```

---

## Why Do Components Need Imports?

Our component doesn't automatically have access to every capability available within the application.

Imports allow us to make additional functionality available to our component.

Conceptually:

```text
Angular / ADF Functionality
           ↓
        Imports
           ↓
       Component
           ↓
        Template
```

As we build more sophisticated components, imports will become increasingly important.

---

# Part 12: Review the Component

Before continuing, let's look at what our component now contains.

```text
Component
├── Metadata
├── Template
├── Properties
├── Methods
├── Lifecycle
└── Imports
```

We've built each of these pieces individually.

Our component has progressed from:

```typescript
export class customDashComponent {

}
```

to a component that:

- Stores information
- Displays information
- Responds to user actions
- Changes the UI
- Runs initialization logic
- Uses additional Angular functionality
- Can be loaded through an Angular route

---

# Part 13: Build the Claims Dashboard

Now that we understand how our component works, we can replace our simple training interface with a more realistic Claims Dashboard.

Open:

```text
customDashComponent.html
```

Replace the training HTML with the Claims Dashboard HTML provided by your instructor.

The new interface will include elements such as:

- Claims summary cards
- Recent claim cases
- Claim statuses
- Featured claim information
- Recent activity
- Dashboard navigation

Save the file and return to:

```text
http://localhost:4200/#/dashboard
```

Your simple training component should now appear as a complete Claims Dashboard.

---

# Part 14: Simplify the Component

The properties and methods we created earlier were used to demonstrate how Angular components work.

They are not currently required by our completed dashboard HTML.

Update `customDashComponent.ts` to:

```typescript
import { Component } from '@angular/core';
import { MatDividerModule } from '@angular/material/divider';
import { HeaderComponent } from '@hxp/shared-hxp/navigation/header';
import { RouterLink } from '@angular/router';

@Component({
    selector: 'hxp-dashboard',
    templateUrl: './customDashComponent.html',
    imports: [MatDividerModule, HeaderComponent, RouterLink],
})
export class customDashComponent {

    constructor() {
    }

    ngOnInit() {

    }
}
```

Save the file.

---

# Lab Complete

You have now created an Angular component from scratch and progressively added functionality to it.

You worked with:

- Component creation
- `@Component` configuration
- HTML templates
- Component properties
- Interpolation
- Event binding
- Component methods
- UI state changes
- Component lifecycle
- Component imports
- Angular routing

Most importantly, you've seen how these pieces work together:

```text
                 Angular Component
                        │
            ┌───────────┴───────────┐
            ▼                       ▼
     Application Logic         HTML Template
            │                       │
            └───────────┬───────────┘
                        ▼
                   User Interface
```

---

# What's Next?

Our Claims Dashboard works, but every user currently receives the same experience.

What if our Custom UI could determine **who the current user is** and change its behavior accordingly?

Next, we'll expand our component functionality to work with:

```text
Component
    ↓
Angular Services
    ↓
Dependency Injection
    ↓
IdentityUserService
    ↓
Current User
    ↓
Group Membership
    ↓
Conditional Logic
    ↓
Angular Routing
    ↓
Different User Experiences
```

This will move us from building an individual Angular component to building a Custom UI that can make decisions and respond to the context of the authenticated user.
