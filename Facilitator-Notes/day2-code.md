## Create the TypeScript Component

### Step 1
**customDashComponent.ts**
```
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

### Step 2
**customDashComponent.ts**
```
// import the component library
import { Component } from '@angular/core';

// add the component decorator that identifies this file as a component
@Component({
    selector: 'hxp-dashboard',
    templateUrl: './customDashComponent.html'
})

// export the class in order to define the component's data and behavior
export class customDashComponent {
    // add a variable called dashboardTitle with the string value of 'Insurance Claims Dashboard'
    dashboardTitle = 'Insurance Claims Dashboard';
}
```
