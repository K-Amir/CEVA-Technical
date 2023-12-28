# CEVA-Technical
# NodeJS
## Exercise 1 (1 points)
```javascript
async function getCountUsers() {
    return {
      total: await got.get("https://my-webservice.moveecar.com/users/count"),
    };
  }
  // Add total from service with 20
  async function computeResult() {
    const result = getCountUsers();
    return result.total + 20;
  }
```
### Is there a problem? 
Yes, there is, because getCountUsers() returns an object with total storing the pending promise without the result. 


## Exercise 2 (1 points)
```javascript
// Call web service and return total vehicles, (got is library to call url)
async function getTotalVehicles() {
    return await got.get('https://my-webservice.moveecar.com/vehicles/total');
}

function getPlurial() {
    let total;
    getTotalVehicles().then(r => total = r);
    if (total <= 0) {
        return 'none';
    }
    if (total <= 10) {
        return 'few';
    }
    return 'many';
}
```
### Is there a problem? 
Yes, when calling getTotalVehicles(), we add it to the callstack but we don't wait for it to finish before checking the value of total.

## Exercise 3 Unit test ( 2 points )
### Write unit tests in jest for the function below in typescript
```javascript
import { expect, test } from '@jest/globals';

function getCapitalizeFirstWord(name: string): string {
  if (name == null) {
    throw new Error('Failed to capitalize first word with null');
  }
  if (!name) {
    return name;
  }
  return name.split(' ').map(
    n => n.length > 1 ? (n.substring(0, 1).toUpperCase() + n.substring(1).toLowerCase()) : n
  ).join(' ');
}

test('1. test', async function () {
    // We test error case
    expect(getCapitalizeFirstWord(null)).toThrow('Failed to capitalize first word with null');

    // We test empty string
    expect(getCapitalizeFirstWord('')).toBe('');

    // We test one letter has to return the same letter because length is 1
    expect(getCapitalizeFirstWord("a")).toBe("a");

    // Happy path
    expect(getCapitalizeFirstWord('happy holidays')).toBe('Happy Holidays');
});
```

# Angular
## Exercise 1: Is there a problem and improve the code (5 points)
```javascript
@Component({
  selector: 'app-users',
  template: `
    <input type="text" [(ngModel)]="query" (ngModelChange)="querySubject.next($event)">
    <div *ngFor="let user of users">
        {{ user.email }}
    </div>
  `
})
export class AppUsers implements OnInit {

  query = '';
  querySubject = new Subject<string>();

  users: { email: string; }[] = [];

  constructor(
    private userService: UserService
  ) {
  }

  ngOnInit(): void {
    concat(
      of(this.query),
      this.querySubject.asObservable()
    ).pipe(
      concatMap(q =>
        timer(0, 60000).pipe(
          this.userService.findUsers(q)
        )
      )
    ).subscribe({
      next: (res) => this.users = res
    });
  }
}
```
### Is there a problem and improve the code (5 points)
query is redundant, if what we want is to give an inital value we can use a BehaviorSubject so we dont need the concat or the of.
The timer makes no sense, we are creating a weird loop, and the pipe() cannot take a function that returns an observable, the pipe can only take
OperatorFunction as type.

### Improved version
```javascript
@Component({
  selector: 'app-users',
  template: `
    <input type="text" [(ngModel)]="query" (ngModelChange)="querySubject.next($event)">
    <div *ngFor="let user of users">
        {{ user.email }}
    </div>
  `
})
export class AppUsers implements OnInit {

  // Removal of query variable
  // ----

  // Replace Subject for BehaviorSubject to add an initial value and trigger the service call after load
  querySubject = new BehaviorSubject<string>("");

  users: { email: string; }[] = [];

  constructor(
    private userService: UserService
  ) {
  }
  ngOnInit(): void {
    this.querySubject.asObservable()
    .pipe(
      // We suscribe to the value of the querySubject
      concatMap(q =>
        // Perform the service call
        this.userService.findUsers(q)
      ),
    ).subscribe({
      // And we suscribe to the result of the service call
      next: (res) => this.users = res
      });
  }
}
```



## Exercise 2 Improve performance (5 points)
```javascript
@Component({
  selector: 'app-users',
  template: `
    <div *ngFor="let user of users">
        {{ getCapitalizeFirstWord(user.name) }}

        // Easy solution
        {{ user.name | titlecase }}
    </div>
  `
})
export class AppUsers {

  @Input()
  users: { name: string; }[];

  constructor() {}
  
//   getCapitalizeFirstWord(name: string): string {
//     return name.split(' ').map(n => n.substring(0, 1).toUpperCase() + n.substring(1).toLowerCase()).join(' ');
//   }
}
```
### Improve performance 
Delete the function and just use the titlecase pipe


## Exercise 3
### Complete and modify AppUserForm class to use Angular Reactive Forms. Add a button to submit.
### The form should return data in this format
```javascript
{
    email: string; // mandatory, must be a email
    name: string; // mandatory, max 128 characters
    birthday?: Date; // Not mandatory, must be less than today
    address: { // mandatory
      zip: number; // mandatory
      city: string; // mandatory, must contains only alpha uppercase and lower and space
    };
}

@Component({
  selector: 'app-user-form',
  template: `
    <form [formGroup]="userForm" (ngSubmit)="doSubmit()">
        <input formControlName="email" type="text" placeholder="email" />
        <input formControlName="name" type="text" placeholder="name" />
        <input formControlName="birthday" type="date" placeholder="birthday" />
        <div formGroupName="address">
            <input formControlName="zip" type="number" placeholder="zip" />
            <input formControlName="city" type="text" placeholder="city" />
        </div>
        <button class="btn btn-primary" type="submit">Submit</button>
    </form>
  `
})
export class AppUserForm {
   
  @Output()
  event = new EventEmitter<{ email: string; name: string; birthday: Date; address: { zip: number; city: string; };}>;

  userForm: FormGroup;
  
  constructor(
    private formBuilder: FormBuilder
  ) { 
    this.userForm = this.formBuilder.group({
      email: ['', [Validators.required, Validators.email]],
      name: ['', [Validators.required, Validators.maxLength(128)]],
      birthday: ['', [this.validateBirthdate()]],
      address: formBuilder.group({
        zip: ['', [Validators.required]],
        city: ['', [Validators.required, Validators.pattern('[a-zA-Z0-9 ]*')]],
      }),
    })
  }

  doSubmit(): void {
    if(this.userForm.valid) {
        this.event.emit(this.userForm.value);
    };
  }

  validateBirthdate() {
      return (control: AbstractControl): ValidationErrors | null => {
        if (control.value === null) return null;
        const selectedDate = new Date(control.value).getTime();
        const currentDate = new Date().getTime();
        return selectedDate > currentDate
          ? {
              dateError: 'Date must be less than today',
            }
          : null;
      };
    }
}
```


# CSS Boostrap
<a href="https://stackblitz.com/edit/web-platform-awrejp?file=index.html" target="_blank">Live demo</a>

## Exercise 1
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN" crossorigin="anonymous">
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL" crossorigin="anonymous"></script>
</head>
<body>
    <div class="container">
        <div class="row mt-4">
            <div class="col-4">
                <div class="bg-gray border-2 shadow-sm border rounded mw-100 p-3 position-relative">
                    <span class="badge rounded-pill bg-danger position-absolute top-0 start-100 translate-middle">+123</span>
                    <h4>Exercice</h4>
                    <p>Redo this card in css ad if possible using bootstrap 4 or 5</p>
                    <div data-id="actions" class="d-flex flex-row-reverse gap-2">
                        <select class="form-select w-auto bg-secondary text-white" aria-label="Options">
                            <option selected>Options</option>
                            <option value="1">works?</option>
                          </select>
                        <div class="btn-group" role="group" aria-label="knowledge buttons">
                            <button type="button" class="btn btn-dark">Got it</button>
                            <button type="button" class="btn btn-outline-dark">I don't know</button>
                        </div>
                       
                    </div>
                </div>
            </div>
        </div>
        
    </div>
</body>
</html>
```

# MongoDB
## Exercise 1
### MongoDb collection users with schema
```javascript
{
    email: string;
    first_name: string;
    last_name: string;
    roles: string[];
    last_connection_date: Date;
}

const text = 'test';
let sixMonthsAgo = new Date();
sixMonthsAgo.setMonth(sixMonthsAgo.getMonth() - 6);

db.collections('users').find(
    {
        email: text,
        {
            $or: [
                { first_name: { $regex: `^${text}` } },
                { last_name: { $regex: `^${text}` } }
            ]
        },
        last_connection_date: {
            $gte: sixMonthsAgo
        }
    }
);


```
### What should be added to the collection so that the query is not slow?
we should add a index on the field last_connection_date
 
## Exercise 2
### MongoDb aggregate (5 points)

```javascript
db.collections('users').aggregate(
    {
        $group: { 
            _id: '$roles',
            users: { $push: '$email' }
         }
    }
)
```

## Exercise 3
### MongoDb update (5 points)
### MongoDb collection users with schema

```javascript
  {
    email: string;
    first_name: string;
    last_name: string;
    roles: string[];
    last_connection_date: Date;
    addresses: {
        zip: number;
        city: string;
    }[]:
  }
```

### Update document ObjectId("5cd96d3ed5d3e20029627d4a"), modify only last_connection_date with current date
```javascript
db.collections('users').updateOne(
    // query
    {
        _id: ObjectId("5cd96d3ed5d3e20029627d4a")
    },
    // update
    {
        $set: {
            last_connection_date: new Date()
        }
    }
);

```

### Update document ObjectId("5cd96d3ed5d3e20029627d4a"), add a role admin
```javascript
db.collections('users').updateOne(
    // query
    {
        _id: ObjectId("5cd96d3ed5d3e20029627d4a")
    },
    // update
    {
        $push: {
            roles: 'admin'
        }
    }
);
```

### Update document ObjectId("5cd96d3ed5d3e20029627d4a"), modify addresses with zip 75001 and replace city with Paris 1
```javascript
db.collections('users').updateOne(
    // query
    {
        _id: ObjectId("5cd96d3ed5d3e20029627d4a"),
        "addresses.zip": 75001
    },
    // update
    {
        $set: {
            "addresses.city": "Paris 1"
        }
    }
);
```


