# Hands-On Lab: Build a Claims Dashboard with Angular and HxP Content Repository

## Overview

In this lab, you will build an Angular-based **Insurance Claims Dashboard** that retrieves live claim information from the HxP Content Repository.

Rather than displaying hard-coded values, the dashboard will query the repository at runtime to discover claim folders and the documents stored within each claim.

By the end of the lab, your dashboard will:

- Connect to the HxP Content Repository using an existing Angular service.
- Query claim folders using HXQL.
- Determine the total number of claims.
- Retrieve the documents associated with each claim.
- Store repository results in TypeScript objects.
- Dynamically generate claim cards using Angular control flow.
- Display claim and document information in a responsive dashboard.
- Handle loading, empty, and error states.

> **Estimated Time:** 2.5–3 hours

---

# What You Will Build

The completed dashboard will represent repository content similar to the following:

```text
/uidev_claims
│
├── CLM-10001
│   ├── ClaimForm.pdf
│   ├── PoliceReport.pdf
│   └── vehicle-damage.jpg
│
├── CLM-10002
│   ├── ClaimForm.pdf
│   └── accident.jpg
│
└── CLM-10003
    └── Estimate.pdf
```

Each folder directly beneath `uidev_claims` represents a **claim**.

The files inside each claim folder represent the **supporting documents for that claim**.

The Angular application will transform this repository structure into a dashboard similar to:

```text
Insurance Claims Dashboard

TOTAL CLAIMS
     7

------------------------------------------------

CLM-10001                     3 Documents

Supporting Documents

    ClaimForm.pdf
    PoliceReport.pdf
    vehicle-damage.jpg

------------------------------------------------

CLM-10002                     2 Documents

Supporting Documents

    ClaimForm.pdf
    accident.jpg
```

The number of claims and documents will be determined dynamically from the repository.

---

# Learning Objectives

After completing this lab, you will be able to:

1. **Integrate** an Angular component with an HxP Content Repository API using Angular dependency injection.

2. **Construct and execute** HXQL queries to retrieve repository folders and content.

3. **Transform and present** repository data dynamically using TypeScript and Angular templates.

---

# Prerequisites

Before beginning this lab, verify that:

- Your Custom UI project runs successfully.
- The `/dashboard` route displays your dashboard component.
- A `uidev_claims` folder exists in the HxP Content Repository.
- `uidev_claims` contains one or more claim folders.
- Claim folders contain test documents such as PDFs or images.

## Generate Our Working Directory and files:
```text
libs/
└── dashboard/
    ├── customDashComponent.ts
    ├── customDashComponent.html
    └── customDashComponent.scss
```
1. Create the `dashboard` directory under _libs_.
2. Create the 3 files above.


---

# Part 1 — Understand the Repository Data Model

Before writing code, consider how the repository content maps to the application.

A repository structure such as:

```text
/uidev_claims
    /CLM-10001
        ClaimForm.pdf
        DamagePhoto.jpg
```

can be represented in TypeScript as:

```text
Claim
 ├── id
 ├── name
 ├── path
 ├── created
 ├── modified
 └── documents[]
        ├── id
        ├── name
        ├── path
        ├── primaryType
        ├── created
        └── modified
```

The dashboard will therefore perform two levels of repository queries:

```text
Query /uidev_claims
        │
        ▼
Find Claim Folders
        │
        ▼
For Each Claim
        │
        ▼
Query Claim Folder
        │
        ▼
Find Documents
        │
        ▼
Build Dashboard
```

This separates **retrieving the data** from **displaying the data**.

---

# Part 2 — Create the Claim Data Models

Open:

```text
libs/dashboard/customDashComponent.ts
```

Begin with the Angular imports:

```typescript
import { Component, inject, OnInit } from '@angular/core';
```

Next, import the HxP Content Repository API types:

```typescript
import {
    Query,
    QueryApi,
    QueryResult
} from '@hylandsoftware/hxcs-js-client';
```

Finally, import the Angular injection token used to access the configured Query API:

```typescript
import { QUERY_API_TOKEN } from '@alfresco/adf-hx-content-services/api';
```

Your imports should now be:

```typescript
import { Component, inject, OnInit } from '@angular/core';
import {
    Query,
    QueryApi,
    QueryResult
} from '@hylandsoftware/hxcs-js-client';
import { QUERY_API_TOKEN } from '@alfresco/adf-hx-content-services/api';
```

---

## Create a Claim Document Interface

Below the imports, create an interface describing the document information needed by the dashboard:

```typescript
interface ClaimDocument {
    id: string;
    name: string;
    path: string;
    primaryType: string;
    created: string;
    modified: string;
}
```

This interface defines the structure the application will use for each claim document.

---

## Create a Claim Interface

Next, create another interface representing a claim:

```typescript
interface Claim {
    id: string;
    name: string;
    path: string;
    created: string;
    modified: string;
    documents: ClaimDocument[];
}
```

Notice that a claim contains:

```typescript
documents: ClaimDocument[];
```

This allows each claim to contain its own collection of supporting documents.

### Checkpoint

At this point you have defined the data structure the dashboard will eventually display.

No repository query has been made yet.

---

# Part 3 — Configure the Dashboard Component

Add the component definition:

```typescript
@Component({
    selector: 'hxp-dashboard',
    templateUrl: './customDashComponent.html',
    styleUrls: ['./customDashComponent.scss']
})
export class customDashComponent implements OnInit {

}
```

The component uses three separate files:

```text
customDashComponent.ts
        │
        ├── Application logic
        │
customDashComponent.html
        │
        ├── User interface
        │
customDashComponent.scss
        │
        └── Presentation and styling
```

This separation makes the component easier to maintain as it grows.

---

# Part 4 — Inject the Repository Query API

The Custom UI application already provides a configured `QueryApi`.

Inside the component class, add:

```typescript
private queryApi = inject<QueryApi>(QUERY_API_TOKEN);
```

This uses Angular dependency injection to request the Query API from the application.

> **Why use dependency injection?**
>
> The application is responsible for configuring the repository API connection. The component requests that existing configured service rather than manually constructing its own API client.

Now add the initial component properties:

```typescript
dashboardTitle = 'Insurance Claims Dashboard';

claims: Claim[] = [];

isLoading = true;
loadError = '';
```

The `claims` array will eventually contain all claim information retrieved from the repository.

---

# Part 5 — Load Data When the Component Starts

Implement Angular's `OnInit` lifecycle hook:

```typescript
ngOnInit(): void {
    this.loadClaims();
}
```

When Angular initializes the dashboard component, it will call:

```typescript
loadClaims()
```

We will create that method next.

---

# Part 6 — Query the Claim Folders

Now that the component has access to the repository's `QueryApi`, we can create a method that retrieves the claim folders.

Each folder directly inside:

```text
/uidev_claims
```

represents one insurance claim.

Our method will:

1. Prepare the component for loading data.
2. Attempt to query the repository.
3. Process the results if the query succeeds.
4. Display an error if something goes wrong.
5. Finish the loading state when the operation is complete.

---

## Create the `loadClaims()` Method

Open:

```text
libs/dashboard/customDashComponent.ts
```

Inside the `customDashComponent` class, below `ngOnInit()`, add:

```typescript
async loadClaims(): Promise<void> {

    this.isLoading = true;
    this.loadError = '';

}
```

Your component should now contain:

```typescript
ngOnInit(): void {
    this.loadClaims();
}

async loadClaims(): Promise<void> {

    this.isLoading = true;
    this.loadError = '';

}
```

### What does `async` mean?

Retrieving information from the Content Repository takes time.

Marking the method as:

```typescript
async
```

allows the method to wait for the repository to respond before continuing.

Later in the method, we will use:

```typescript
await
```

to wait for that response.

---

## Add Error Handling

Repository requests can fail for many reasons, including connectivity, permissions, or an invalid query.

We do not want an error to cause the dashboard to fail without providing useful feedback.

Inside `loadClaims()`, add a `try`, `catch`, and `finally` structure:

```typescript
async loadClaims(): Promise<void> {

    this.isLoading = true;
    this.loadError = '';

    try {

        // We will query the repository here.

    } catch (error) {

        console.error(
            'Unable to load claims:',
            error
        );

        this.loadError =
            'Unable to retrieve claims from the Content Repository.';

    } finally {

        this.isLoading = false;

    }
}
```

### Understanding `try`, `catch`, and `finally`

The `try` block contains the code we want the application to attempt:

```typescript
try {

    // Attempt repository operations

}
```

If something inside `try` fails, execution moves to `catch`:

```typescript
catch (error) {

    // Handle the error

}
```

The `finally` block runs after the operation finishes, regardless of whether it succeeded or failed:

```typescript
finally {

    this.isLoading = false;

}
```

In our dashboard, this gives us a useful pattern:

```text
Start Loading
     │
     ▼
    try
     │
     ├──── Success ────► Process Repository Data
     │
     └──── Failure ────► catch
                              │
                              ▼
                         Display Error
     │
     ▼
  finally
     │
     ▼
Stop Loading
```

> **Automate Connection**
>
> This should look familiar if you have used Try/Catch logic when scripting in an Automate process. The same general idea applies here: attempt an operation and provide controlled behavior when that operation fails.

---

## Create the HXQL Query

Now we have a place to perform our repository operation.

Inside the `try` block, replace:

```typescript
// We will query the repository here.
```

with:

```typescript
const claimQuery: Query = {
    query: `
        SELECT *
        FROM SysFolder
        WHERE sys_parentPath = '/uidev_claims'
    `,
    limit: 1000,
    offset: 0
};
```

Your `try` block should now look like:

```typescript
try {

    const claimQuery: Query = {
        query: `
            SELECT *
            FROM SysFolder
            WHERE sys_parentPath = '/uidev_claims'
        `,
        limit: 1000,
        offset: 0
    };

}
```

---

## Understanding the Query Object

We created a variable named:

```typescript
claimQuery
```

and defined it as a:

```typescript
Query
```

The `Query` type comes from the HxP Content Repository client that we imported earlier.

Our object contains three properties:

```typescript
const claimQuery: Query = {
    query: `...`,
    limit: 1000,
    offset: 0
};
```

### `query`

The `query` property contains the HXQL statement that determines what repository content we want to retrieve.

```sql
SELECT *
FROM SysFolder
WHERE sys_parentPath = '/uidev_claims'
```

Let's break that down.

### `SELECT *`

```sql
SELECT *
```

requests the available properties for matching repository objects.

### `FROM SysFolder`

```sql
FROM SysFolder
```

limits the results to folders.

This is important because each folder beneath `uidev_claims` represents a claim.

### `WHERE sys_parentPath = '/uidev_claims'`

```sql
WHERE sys_parentPath = '/uidev_claims'
```

limits the results to folders whose immediate parent is:

```text
/uidev_claims
```

For example, given:

```text
/uidev_claims
    /CLM-10001
    /CLM-10002
    /CLM-10003
```

the query returns those three claim folders.

### `limit`

```typescript
limit: 1000
```

specifies the maximum number of results requested.

### `offset`

```typescript
offset: 0
```

instructs the query to begin with the first available result.

---

## Execute the Repository Query

Creating `claimQuery` only describes what we want to retrieve.

We still need to send that query to the repository.

Immediately after the `claimQuery` object, add:

```typescript
const response =
    await this.queryApi.getDocumentsByQuery(claimQuery);
```

The `await` keyword tells the method:

> Wait for the Content Repository to respond before continuing.

Our code now performs the following:

```text
claimQuery
     │
     ▼
QueryApi
     │
     ▼
HxP Content Repository
     │
     ▼
Repository Response
```

---

## Get the Query Results

The repository response contains more than just the documents themselves.

First, retrieve the query result:

```typescript
const result: QueryResult = response.data;
```

Then retrieve the documents returned by the query:

```typescript
const claimFolders = result.documents ?? [];
```

The `?? []` means:

> If `result.documents` does not contain a value, use an empty array instead.

This ensures `claimFolders` can still be safely processed even when the repository returns no matching claims.

---

## View the Results

Before building the dashboard, let's verify that our query works.

Add:

```typescript
console.log(
    `Found ${claimFolders.length} claim folders.`,
    claimFolders
);

this.claims = claimFolders.map((folder: any) => ({
    id: folder.sys_id,
    name: folder.sys_name,
    path: folder.sys_path,
    created: folder.sys_created,
    modified: folder.sys_modified,
    documents: []
}));
```

Your `try` block should now contain:

```typescript
try {

    const claimQuery: Query = {
        query: `
            SELECT *
            FROM SysFolder
            WHERE sys_parentPath = '/uidev_claims'
        `,
        limit: 1000,
        offset: 0
    };

    const response =
        await this.queryApi.getDocumentsByQuery(claimQuery);

    const result: QueryResult = response.data;

    const claimFolders = result.documents ?? [];

    console.log(
        `Found ${claimFolders.length} claim folders.`,
        claimFolders
    );

}
```

---

## Test the Repository Query

Save the component and reload the dashboard.

Open your browser's developer tools and select the **Console**.

You should see a message similar to:

```text
Found 7 claim folders.
```

Your number may be different depending on the number of claim folders currently stored in the repository.

Expand the returned array in the console.

Each claim folder should contain repository properties such as:

```text
sys_id
sys_name
sys_parentPath
sys_path
sys_primaryType
sys_created
sys_modified
```

For example:

```text
sys_name: "CLM-10001"
sys_parentPath: "/uidev_claims"
sys_path: "/uidev_claims/CLM-10001"
sys_primaryType: "SysFolder"
```

### Checkpoint

Before continuing, verify that:

- [ ] The application compiles without errors.
- [ ] The dashboard loads.
- [ ] The repository query completes successfully.
- [ ] The console displays the expected number of claim folders.
- [ ] The returned objects represent folders inside `/uidev_claims`.

If those checks pass, your Angular component is now successfully retrieving live data from the HxP Content Repository.

---

# Part 7 — Display the Total Number of Claims

Now connect the repository data to the HTML.

Open:

```text
libs/dashboard/customDashComponent.html
```

Add:

```html
<div class="dashboard">

    <h1>{{ dashboardTitle }}</h1>

    <h2>Total Claims</h2>

    <div>
        {{ claims.length }}
    </div>

</div>
```

Notice that no claim count is hard-coded.

Angular evaluates:

```html
{{ claims.length }}
```

using the array populated by the repository query.

If another claim folder is created, the number changes the next time the dashboard retrieves repository data.

### Checkpoint

Reload the dashboard.

Verify that **Total Claims** matches the number of folders stored directly inside:

```text
/uidev_claims
```

---

# Part 8 — Dynamically Generate a Card for Every Claim

Now that `claims` contains repository data, Angular can dynamically generate UI elements from the array.

Replace the current HTML with:

```html
<div class="dashboard">

    <h1>{{ dashboardTitle }}</h1>

    <h2>Total Claims: {{ claims.length }}</h2>

    <div class="claims-grid">

        @for (claim of claims; track claim.id) {

            <div class="claim-card">

                <h2>
                    {{ claim.name }}
                </h2>

                <p>
                    {{ claim.documents.length }} Documents
                </p>

            </div>

        }

    </div>

</div>
```

The Angular control flow:

```html
@for (claim of claims; track claim.id)
```

creates one block of HTML for every object in the `claims` array.

The repository now determines how many claim cards appear.

### Checkpoint

Reload the dashboard.

You should see one card for every claim folder.

At this stage every card should report:

```text
0 Documents
```

That's expected.

We have discovered the claims, but we have not queried the files inside them yet.

---

# Part 9 — Retrieve Documents for a Claim

Return to:

```text
customDashComponent.ts
```

Create a new method below `loadClaims()`:

```typescript
private async loadClaimDocuments(
    claimPath: string
): Promise<ClaimDocument[]> {

}
```

This method receives the path of a claim.

For example:

```text
/uidev_claims/CLM-10001
```

---

## Query the Claim Folder

Inside the method, create another repository query:

```typescript
const documentQuery: Query = {
    query: `
        SELECT *
        FROM SysContent
        WHERE sys_parentPath = '${claimPath}'
    `,
    limit: 1000,
    offset: 0
};
```

Unlike the previous query, this query requests:

```sql
FROM SysContent
```

because we want the files stored inside the claim folder.

Execute the query:

```typescript
const response =
    await this.queryApi.getDocumentsByQuery(documentQuery);
```

Retrieve the results:

```typescript
const result: QueryResult = response.data;

const documents = result.documents ?? [];
```

---

## Transform Repository Documents

The repository returns objects containing system properties such as:

```text
sys_id
sys_name
sys_path
sys_primaryType
sys_created
sys_modified
```

Transform those values into our `ClaimDocument` interface:

```typescript
return documents.map((document: any) => {

    return {
        id: document.sys_id,
        name: document.sys_name,
        path: document.sys_path,
        primaryType: document.sys_primaryType,
        created: document.sys_created,
        modified: document.sys_modified
    };

});
```

The completed method should be:

```typescript
private async loadClaimDocuments(
    claimPath: string
): Promise<ClaimDocument[]> {

    const documentQuery: Query = {
        query: `
            SELECT *
            FROM SysContent
            WHERE sys_parentPath = '${claimPath}'
        `,
        limit: 1000,
        offset: 0
    };

    const response =
        await this.queryApi.getDocumentsByQuery(documentQuery);

    const result: QueryResult = response.data;

    const documents = result.documents ?? [];

    return documents.map((document: any) => {

        return {
            id: document.sys_id,
            name: document.sys_name,
            path: document.sys_path,
            primaryType: document.sys_primaryType,
            created: document.sys_created,
            modified: document.sys_modified
        };

    });
}
```

---

# Part 10 — Load Documents for Every Claim

We now have two capabilities:

```text
loadClaims()
    → Finds claims

loadClaimDocuments()
    → Finds documents inside one claim
```

Next, connect them.

Return to `loadClaims()`.

Find the temporary code:

```typescript
this.claims = claimFolders.map((folder: any) => ({
    id: folder.sys_id,
    name: folder.sys_name,
    path: folder.sys_path,
    created: folder.sys_created,
    modified: folder.sys_modified,
    documents: []
}));
```

Remove it.

Replace it with:

```typescript
this.claims = await Promise.all(

    claimFolders.map(async (folder: any) => {

        const documents =
            await this.loadClaimDocuments(folder.sys_path);

        return {
            id: folder.sys_id,
            name: folder.sys_name,
            path: folder.sys_path,
            created: folder.sys_created,
            modified: folder.sys_modified,
            documents: documents
        };

    })

);
```

`Promise.all()` allows the application to retrieve the document collections for the claim folders without processing each request strictly one at a time.

The resulting `claims` array now contains both levels of information:

```text
claims
│
├── Claim
│   ├── name
│   ├── path
│   └── documents
│       ├── Document
│       ├── Document
│       └── Document
│
├── Claim
│   ├── name
│   └── documents
│       └── Document
│
└── ...
```

---

## Sort the Claims

After `Promise.all()`, add:

```typescript
this.claims.sort((a, b) =>
    a.name.localeCompare(b.name)
);
```

This keeps the dashboard presentation predictable regardless of the order returned by the repository.

---

# Part 11 — Display Claim Documents

Return to:

```text
customDashComponent.html
```

Inside each claim card, add another Angular `@for`:

```html
@for (claim of claims; track claim.id) {

    <div class="claim-card">

        <h2>
            {{ claim.name }}
        </h2>

        <p>
            {{ claim.documents.length }} Documents
        </p>

        @for (
            document of claim.documents;
            track document.id
        ) {

            <div class="document-row">

                <span>
                    📄
                </span>

                <span>
                    {{ document.name }}
                </span>

            </div>

        }

    </div>

}
```

We now have **nested dynamic content**:

```text
@for each Claim
        │
        └── @for each Document
```

The first loop generates claim cards.

The second loop generates the documents belonging to that claim.

### Checkpoint

Reload the dashboard.

Verify:

- The correct number of claims appears.
- Each claim appears once.
- The correct number of documents appears for each claim.
- File names are displayed beneath the correct claim.

---

# Part 12 — Handle Claims Without Documents

A claim might exist before supporting documentation has been added.

Inside the document section, add:

```html
@if (claim.documents.length === 0) {

    <div class="no-documents">

        No documents have been
        added to this claim.

    </div>

}
```

Then display documents with:

```html
@for (
    document of claim.documents;
    track document.id
) {

    <div class="document-row">

        <div class="document-icon">
            📄
        </div>

        <div class="document-details">

            <div class="document-name">
                {{ document.name }}
            </div>

            <div class="document-type">
                {{ document.primaryType }}
            </div>

        </div>

    </div>

}
```

This provides useful feedback instead of displaying an empty card.

---

# Part 13 — Add Loading and Error States

Repository data takes time to retrieve.

The component already contains:

```typescript
isLoading = true;
loadError = '';
```

And `loadClaims()` sets:

```typescript
this.isLoading = true;
```

before retrieving data.

The `finally` block sets:

```typescript
this.isLoading = false;
```

after the request completes.

Use those values in the template:

```html
@if (isLoading) {

    <div class="message-card">

        <h2>Loading Claims</h2>

        <p>
            Retrieving claim information from the
            Content Repository...
        </p>

    </div>

} @else if (loadError) {

    <div class="message-card error-card">

        <h2>Unable to Load Claims</h2>

        <p>
            {{ loadError }}
        </p>

    </div>

} @else {

    <!-- Claims dashboard -->

}
```

This creates three possible application states:

```text
Loading
   │
   ├── Success → Display Claims
   │
   └── Failure → Display Error
```

---

# Part 14 — Complete the TypeScript Component

Your completed `customDashComponent.ts` should now resemble:

```typescript
import { Component, inject, OnInit } from '@angular/core';
import {
    Query,
    QueryApi,
    QueryResult
} from '@hylandsoftware/hxcs-js-client';
import { QUERY_API_TOKEN } from '@alfresco/adf-hx-content-services/api';

interface ClaimDocument {
    id: string;
    name: string;
    path: string;
    primaryType: string;
    created: string;
    modified: string;
}

interface Claim {
    id: string;
    name: string;
    path: string;
    created: string;
    modified: string;
    documents: ClaimDocument[];
}

@Component({
    selector: 'hxp-dashboard',
    templateUrl: './customDashComponent.html',
    styleUrls: ['./customDashComponent.scss']
})
export class customDashComponent implements OnInit {

    private queryApi = inject<QueryApi>(QUERY_API_TOKEN);

    dashboardTitle = 'Insurance Claims Dashboard';

    claims: Claim[] = [];

    isLoading = true;
    loadError = '';

    ngOnInit(): void {
        this.loadClaims();
    }

    async loadClaims(): Promise<void> {

        this.isLoading = true;
        this.loadError = '';

        try {

            const claimQuery: Query = {
                query: `
                    SELECT *
                    FROM SysFolder
                    WHERE sys_parentPath = '/uidev_claims'
                `,
                limit: 1000,
                offset: 0
            };

            const response =
                await this.queryApi.getDocumentsByQuery(claimQuery);

            const result: QueryResult = response.data;

            const claimFolders = result.documents ?? [];

            console.log(
                `Found ${claimFolders.length} claim folders.`,
                claimFolders
            );

            this.claims = await Promise.all(

                claimFolders.map(async (folder: any) => {

                    const documents =
                        await this.loadClaimDocuments(folder.sys_path);

                    return {
                        id: folder.sys_id,
                        name: folder.sys_name,
                        path: folder.sys_path,
                        created: folder.sys_created,
                        modified: folder.sys_modified,
                        documents: documents
                    };

                })

            );

            this.claims.sort((a, b) =>
                a.name.localeCompare(b.name)
            );

        } catch (error) {

            console.error(
                'Unable to load claims:',
                error
            );

            this.loadError =
                'Unable to retrieve claims from the Content Repository.';

        } finally {

            this.isLoading = false;

        }
    }


    private async loadClaimDocuments(
        claimPath: string
    ): Promise<ClaimDocument[]> {

        const documentQuery: Query = {
            query: `
                SELECT *
                FROM SysContent
                WHERE sys_parentPath = '${claimPath}'
            `,
            limit: 1000,
            offset: 0
        };

        const response =
            await this.queryApi.getDocumentsByQuery(documentQuery);

        const result: QueryResult = response.data;

        const documents = result.documents ?? [];

        return documents.map((document: any) => {

            return {
                id: document.sys_id,
                name: document.sys_name,
                path: document.sys_path,
                primaryType: document.sys_primaryType,
                created: document.sys_created,
                modified: document.sys_modified
            };

        });
    }
}
```

---

# Part 15 — Build the Finished Dashboard Template

Now replace the working test HTML with the finished dashboard structure:

```html
<div class="dashboard">

    <div class="dashboard-header">

        <div>
            <div class="eyebrow">
                CLAIMS MANAGEMENT
            </div>

            <h1>
                {{ dashboardTitle }}
            </h1>

            <p class="subtitle">
                Claims and supporting documents from the
                HxP Content Repository.
            </p>
        </div>

        <div class="total-claims-card">

            <div class="total-label">
                TOTAL CLAIMS
            </div>

            <div class="total-number">
                {{ claims.length }}
            </div>

            <div class="total-description">
                Claims in repository
            </div>

        </div>

    </div>


    @if (isLoading) {

        <div class="message-card">

            <div class="loading-spinner"></div>

            <h2>Loading Claims</h2>

            <p>
                Retrieving claim information from the
                Content Repository...
            </p>

        </div>

    } @else if (loadError) {

        <div class="message-card error-card">

            <h2>Unable to Load Claims</h2>

            <p>
                {{ loadError }}
            </p>

        </div>

    } @else {

        <div class="section-header">

            <div>
                <h2>Claims</h2>

                <p>
                    Review the supporting documents
                    associated with each claim.
                </p>
            </div>

            <div class="claim-count">
                {{ claims.length }} Claims
            </div>

        </div>


        @if (claims.length === 0) {

            <div class="message-card">

                <h2>No Claims Found</h2>

                <p>
                    There are currently no claims in the
                    uidev_claims repository folder.
                </p>

            </div>

        }


        <div class="claims-grid">

            @for (claim of claims; track claim.id) {

                <div class="claim-card">

                    <div class="claim-header">

                        <div>

                            <div class="claim-label">
                                CLAIM
                            </div>

                            <h2>
                                {{ claim.name }}
                            </h2>

                        </div>

                        <div class="document-count">

                            <strong>
                                {{ claim.documents.length }}
                            </strong>

                            <span>
                                Documents
                            </span>

                        </div>

                    </div>


                    <div class="documents-header">
                        Supporting Documents
                    </div>


                    <div class="documents">

                        @if (claim.documents.length === 0) {

                            <div class="no-documents">
                                No documents have been
                                added to this claim.
                            </div>

                        }


                        @for (
                            document of claim.documents;
                            track document.id
                        ) {

                            <div class="document-row">

                                <div class="document-icon">
                                    📄
                                </div>

                                <div class="document-details">

                                    <div class="document-name">
                                        {{ document.name }}
                                    </div>

                                    <div class="document-type">
                                        {{ document.primaryType }}
                                    </div>

                                </div>

                            </div>

                        }

                    </div>


                    <div class="claim-footer">

                        {{ claim.documents.length }}
                        supporting document(s)

                    </div>

                </div>

            }

        </div>

    }

</div>
```

---

# Part 16 — Style the Dashboard

Open:

```text
customDashComponent.scss
```

Add the following styles:

```scss
:host {
    display: block;
    background: #f5f7fa;
    min-height: 100vh;
}

.dashboard {
    max-width: 1400px;
    margin: 0 auto;
    padding: 40px;
    font-family: Arial, Helvetica, sans-serif;
    color: #1f2937;
}

.dashboard-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 30px;
    margin-bottom: 45px;
}

.eyebrow {
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 1.6px;
    color: #667085;
    margin-bottom: 8px;
}

.dashboard-header h1 {
    margin: 0;
    font-size: 36px;
    font-weight: 700;
    color: #172b4d;
}

.subtitle {
    margin-top: 10px;
    color: #667085;
    font-size: 16px;
}

.total-claims-card {
    background: white;
    min-width: 180px;
    padding: 22px 28px;
    border-radius: 12px;
    border: 1px solid #e3e7ed;
    box-shadow: 0 3px 10px rgba(0, 0, 0, 0.05);
}

.total-label {
    font-size: 11px;
    font-weight: 700;
    letter-spacing: 1.2px;
    color: #667085;
}

.total-number {
    font-size: 42px;
    line-height: 1;
    font-weight: 700;
    color: #172b4d;
    margin: 8px 0;
}

.total-description {
    font-size: 13px;
    color: #667085;
}

.section-header {
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    margin-bottom: 20px;
}

.section-header h2 {
    margin: 0;
    font-size: 24px;
    color: #172b4d;
}

.section-header p {
    margin: 6px 0 0;
    color: #667085;
}

.claim-count {
    background: #e9eef6;
    padding: 7px 13px;
    border-radius: 20px;
    font-size: 13px;
    font-weight: 600;
    color: #344054;
}

.claims-grid {
    display: grid;
    grid-template-columns:
        repeat(auto-fit, minmax(360px, 1fr));
    gap: 22px;
}

.claim-card {
    background: white;
    border: 1px solid #e1e6ed;
    border-radius: 12px;
    overflow: hidden;
    box-shadow:
        0 2px 7px rgba(0, 0, 0, 0.04);
    transition:
        transform 0.15s ease,
        box-shadow 0.15s ease;
}

.claim-card:hover {
    transform: translateY(-2px);
    box-shadow:
        0 7px 18px rgba(0, 0, 0, 0.08);
}

.claim-header {
    padding: 22px;
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    border-bottom: 1px solid #edf0f4;
}

.claim-label {
    font-size: 10px;
    letter-spacing: 1.4px;
    font-weight: 700;
    color: #667085;
}

.claim-header h2 {
    margin: 5px 0 0;
    font-size: 20px;
    color: #172b4d;
}

.document-count {
    text-align: center;
    background: #f2f5f9;
    border-radius: 8px;
    padding: 8px 12px;
}

.document-count strong {
    display: block;
    font-size: 18px;
    color: #172b4d;
}

.document-count span {
    display: block;
    font-size: 10px;
    color: #667085;
    margin-top: 2px;
}

.documents-header {
    padding: 14px 22px 8px;
    font-size: 11px;
    font-weight: 700;
    letter-spacing: 0.7px;
    color: #667085;
}

.documents {
    padding: 0 14px 14px;
}

.document-row {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 11px 8px;
    border-radius: 7px;
}

.document-row:hover {
    background: #f6f8fb;
}

.document-icon {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 36px;
    height: 36px;
    background: #eef2f7;
    border-radius: 7px;
    flex-shrink: 0;
}

.document-details {
    min-width: 0;
}

.document-name {
    font-size: 14px;
    font-weight: 600;
    color: #344054;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.document-type {
    margin-top: 3px;
    font-size: 11px;
    color: #98a2b3;
}

.no-documents {
    padding: 20px 8px;
    color: #98a2b3;
    font-size: 13px;
    text-align: center;
}

.claim-footer {
    padding: 12px 22px;
    background: #fafbfc;
    border-top: 1px solid #edf0f4;
    color: #667085;
    font-size: 11px;
}

.message-card {
    background: white;
    border: 1px solid #e1e6ed;
    border-radius: 12px;
    padding: 50px;
    text-align: center;
}

.message-card h2 {
    margin: 10px 0;
    color: #172b4d;
}

.message-card p {
    color: #667085;
}

.error-card {
    border-color: #f1b5b5;
}

.loading-spinner {
    width: 32px;
    height: 32px;
    margin: 0 auto 15px;
    border: 3px solid #e4e7ec;
    border-top-color: #475467;
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
}

@keyframes spin {
    to {
        transform: rotate(360deg);
    }
}

@media (max-width: 700px) {

    .dashboard {
        padding: 20px;
    }

    .dashboard-header {
        flex-direction: column;
        align-items: stretch;
    }

    .total-claims-card {
        width: auto;
    }

    .claims-grid {
        grid-template-columns: 1fr;
    }

    .section-header {
        align-items: flex-start;
        flex-direction: column;
        gap: 12px;
    }
}
```

---

# Part 17 — Test the Completed Dashboard

Save all three files and allow the Angular development server to rebuild the application.

Navigate to:

```text
http://localhost:4200/#/dashboard
```

Verify the following:

## Testing Checklist

- [ ] The dashboard loads without compilation errors.
- [ ] The dashboard retrieves claims from `/uidev_claims`.
- [ ] **Total Claims** matches the number of claim folders in the repository.
- [ ] Each claim folder generates its own dashboard card.
- [ ] Each card displays the claim folder name.
- [ ] Each card displays the correct number of supporting documents.
- [ ] Document filenames appear beneath the correct claim.
- [ ] A claim without documents displays the empty-document message.
- [ ] The loading state appears while repository data is being retrieved.
- [ ] No repository or Angular errors appear in the browser console.

---

# Understanding the Completed Application

The completed application follows this flow:

```text
Angular loads customDashComponent
            │
            ▼
         ngOnInit()
            │
            ▼
        loadClaims()
            │
            ▼
      HxP QueryApi
            │
            ▼
SELECT folders beneath
     /uidev_claims
            │
            ▼
      Claim Folders
            │
            ▼
   For Each Claim Folder
            │
            ▼
   loadClaimDocuments()
            │
            ▼
SELECT content beneath
     the claim path
            │
            ▼
      Claim Documents
            │
            ▼
       claims[]
            │
            ▼
 Angular Template @for
            │
            ▼
   Claims Dashboard
```

The important architectural concept is that the **repository is the source of truth**.

The application does not know beforehand:

- How many claims exist.
- What the claim numbers are.
- How many documents belong to a claim.
- What the document filenames are.

Those values are discovered at runtime.

---

# Key Concepts

## Dependency Injection

The component obtains the repository API through:

```typescript
private queryApi = inject<QueryApi>(QUERY_API_TOKEN);
```

This allows the component to use the Query API already configured by the application.

---

## HXQL

HXQL is used to determine what repository content should be returned.

Claims are retrieved with:

```sql
SELECT *
FROM SysFolder
WHERE sys_parentPath = '/uidev_claims'
```

Documents are retrieved with:

```sql
SELECT *
FROM SysContent
WHERE sys_parentPath = '/uidev_claims/CLAIM'
```

---

## Component State

Repository results are stored in:

```typescript
claims: Claim[] = [];
```

Once the array changes, Angular updates the template.

---

## Interpolation

Values are displayed using interpolation:

```html
{{ claims.length }}
```

and:

```html
{{ document.name }}
```

---

## Angular Control Flow

Conditional content is displayed using:

```html
@if (...)
```

Collections are rendered using:

```html
@for (...)
```

This allows the dashboard UI to respond to repository data rather than requiring hard-coded HTML for every claim.

---

# Challenge

If time permits, consider how you might extend the dashboard.

For example:

- Make a document filename clickable.
- Open a document from the repository.
- Display different icons for PDFs and images.
- Display the date a claim was created.
- Sort claims by creation date.
- Add a search field for claim numbers.
- Add a document total across all claims.
- Refresh the dashboard without reloading the page.

> These features are not required to complete the lab. They demonstrate how the same repository data can support increasingly sophisticated Custom UI experiences.

---

# Lab Complete

You have built a dynamic Angular dashboard that connects directly to the HxP Content Repository.

The completed solution combines several Angular and Automate Custom UI concepts:

```text
Angular Components
       +
Dependency Injection
       +
TypeScript Interfaces
       +
Async API Calls
       +
HxP QueryApi
       +
HXQL
       +
Component State
       +
@if / @for
       +
HTML / SCSS
       =
Dynamic Custom UI
```

Rather than creating a static page, you have created an application that **discovers repository content and transforms that data into a usable interface**.

This same pattern can be applied to many other Custom UI requirements where Automate processes create or manage content that users need to review through a purpose-built interface.
