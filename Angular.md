# FoundationAngular
## Commands

Generate a new Angular application:
`ng new [app name] --standalone=false`

Add additional libraries:
`npm install --save [module]`

Start development server:
`ng serve`

Build final application: 
`ng build`

Generate components:
`ng generate component [path/to/file/name]`
`ng g c [name]`

Generate services:
`ng g s [path/to/service]`

Generate new modules:
`ng g m [module name]`

Progressive Web Application:
`ng add @angular/pwa`

## How it works
```typescript
@NgModule({
	delcarations: [ AppComponent ],
	imports: [ BrowserModule ],
	exports: [ ],
	providers: [ ],
	bootstrap: [ AppComponent ]
})
export class AppModule { }
```

|Property |Purpose |
|---------|--------|
|declarations| make a component available within the module|
|imports| make exported components from another module available within this module
|exports| make components, among others, from this module available within other modules
|providers| provide a service to all components and services within this module
|bootstrap| the component to bootstrap if this module is bootstrapped/started

## Accessing data

### From component to template
1. `{{ property }}`
2. Value binding `[attribute]="property"`

### From template to component
1. Event binding `(change)="function($event)"`

### Between components
- Use `@Input()` to push in data.
	- Other component: `[property]="some value"`
```typescript
@Input(
	{
		alias: 'changedName',
		required: true,
		transform: val => !!val //change the value before assignment
	}
)
```
- Use `@Output()` along with `new Subject<type>()` to send out data. 
	- Other component: `(subject)="ownMethodCall($event)`

## Directives
*Examples: ngFor, ngIf, ngClass, ngSwitch*

**`ngClass` for conditional styling**
```html
<input type="text" [disabled]="isDisabled"
	[ngClass]="{ 'grey-border': isDisabled, 
		'blue-border': !isDisabled }">

<td [ngClass]="val > 10 ? 'red' : 'green'">{{ val}}</td>
```

**`ngIf` -- Conditional Display**
```html
<div *ngIf="cart.length > 0; else elseBlock">
	Content shown when condition is true
</div>

<ng-template #elseBlock> <!-- elseBlock is template reference -->
	Content shown when condition is false
</ng-template>
```

Angular's new flow control:
```html
@if (expression) {
	<div>True block</div>
} @else {
	<div>False block</div>
}
```

**`ngFor` -- Loops**
```html
<ul>
	<li *ngFor="let item of cart; let i = index;">
		{{item}}
	</li>
</ul>
```

```html
@for (t of texts; let idx = $index; track t) {
}
```

**`ngSwitch` -- Multi condition**
```ts
selected: string;

update(event) {
	this.selected = event.target.value;
}
```
```html
<select (change)="update($event)">
	<option *ngFor="d of deals" [value]="d">{{d}}</option>
</select>
...

<div [ngSwitch]="selected">
	<div *ngSwitchCase="a">...</div>
	<div *ngSwitchCase="b">...</div>
	<div *ngSwitchDefault>...for no match...</div>
</div>
```

Updated syntax:
```ts
@switch(condition) {
	@case (caseA) {
		Case A.
	}
	@case (caseB) {
		Case B.
	}
	@default {
		Default case.
	}
}
```

**Pipes**
```html
{{n | titlecase}}
*ngFor="let n of names | titlecase"
{{ value | number: '1.1-3' | currency: 'SGD':'symbol-narrow' }}
```
Examples:
1. uppercase
2. titlecase
3. percent
4. number (e.g. `number:'1.1-3'`)
5. slice (e.g. `slice:1:3`)
6. json
7. keyvalue (iterate through an object, and access properties using `ele.key` and `ele.value`
8. i18nselect (e.g. `{{ gender | i18nselect: {'male': 'him', 'female': 'her', ...}}`

# Forms
Add `ReactiveFormsModule` into the module imports. 

## Basic structure
```ts
fb: FormBuilder = inject(FormBuilder);
form!: FormGroup;
labels = ['A', 'B', 'C' ];

ngOnInit() {
	this.array = this.fb.array(
		this.labels.map(() => this.fb.control<string>(''))
	)
	this.form = this.fb.group({
		name: this.fb.control<string>(''),
		email: this.fb.control<string>(''),
		attending: this.fb.control<string>(''),
		demoArray: this.array
	})
}
```

```html
<form [formGroup]="form" (submit)="processForm()">
	Name: <input type="text" formControlName="name">
	Email: <input type="text" formControlName="email">
	Attending:
	<input type="radio" value="yes" formControlName="attending">Yes
	<input type="radio" value="no" formControlName="attending">No
</form>
```

## Dynamic addition of array
```html
<form [formGroup]="form" (submit)="processForm()">
	<button type="button">Add</button>
	
	<tbody>
		@for(row of array.controls; let i = $index; track row) {
		<tr [formGroupName]='i'>
			<td> <input type="date" formControlName="date"> </td>
			<td> <input type="text" formControlName="description"> </td>
		</tr>
		}
	</tbody>
</form>
```
```ts
form: FormGroup;
array: FormArray;

ngOnInit() {
	this.array = this.fb.array([]);
	this.form = this.fb.group({todos: this.array })
}

addNewRow() {
	const rowGroup = this.fb.group({
		date: this.fb.control<Date>(new Date()),
		description: this.fb.control<string>('')
	})
	this.array.push(rowGrop);
}

/**
Note that there is some issue if you track by index, and try to remove the element by index. 
If you do that, try resetting the form by using
this.form.setControl('arrayRef', array)
*/
removeRow(idx: number) {
	this.array.removeAt(idx);
}
```

## Reading form values and data
```ts
const data = this.form.value as RSVP 
// use whatever model the form maps to
```
Variable names in the form and the model need to be the same to automatically map values. Otherwise, need to do manual mapping. 

## Validation
Pre-built validators:
1. `required` -- mandatory field
2. `requiredTrue` -- requires a checkbox to be checked
3. `email` -- entry must be in a valid email format
4. `min`, `max` -- validate a number range
5. `minLength`, `maxLength`
6. `pattern` -- use regular expression to validate a field

```ts
this.form = this.fb.group({
	name: this.fb.control<string>('', [Validators.required]),
	...
})
```
### Checking form and form field validity
Use `.valid` or `invalid` on `FormGroup` or `FormControl`
e.g. `form.get('email').valid`

To check if a field has a specific type of error:
`form.get('email').hasError('email')`

### Error messages
**Conditional display:**
```html
<div *ngIf="form.get('name').hasError('required')">
	Please enter your name
</div>
```

**Disabling button:**
```html
<button type="submit" [disabled]="form.invalid">
	Submit
</button>
```
**Additional conditions:**
To ensure that the form fields have been touched or received any input from the user before displaying the error message, you can use `pristine` or `touched` (and their counterparts `dirty` and `untouched`). 
```ts
// If the field is pristine, it will not be marked as invalid.
protected isControlValid(field: string): boolean {
	return !!this.form.get(field)?.valid || this.form.get(field)?.pristine;
}
```
## Custom validators
```ts
const nonWhitespace = (ctrl: AbstractControl) => {
	if (ctrl.value.trim().length > 0)
		return (null) //return null if there is no error
	return {nonWhitespace: true} as ValidationErrors
	// return an object indicating what errors have occurred
}
```

# Content projection

In child:
```html
<form ...>
	<!-- content to be projected below -->
	<ng-content></ng-content>
</form>
```
```ts
export class RegistrationComponent implments OnInit {
	@Input() title: string

	get values(): Registration {
		return this.form.value as Registration
	}
	...
}
```

In parent:
```html
<app-child #regForm>
	<button type="button" (click)="process()">
		Register
	</button>
</app-child>
```

To gain a reference to the child component instance:
```ts
export class AppComponent implements AfterViewInit {
	// Query wth class
	@ViewChild(RegistrationComponent)
	regForm: RegistrationComponent

	// Or query with template reference
	@ViewChild('regForm')
	regForm: RegistrationComponent

	ngAfterViewInit() { 
		// Instead of using attribute binding, programmatically set the component attribute.
		this.regForm.title = "New product"
	}

	/**
	* When the button (projected is pressed, get the values from the component
	*/
	process() {
		const newReg: Registration = this.regForm.values
		...
	}
}
```

# Progressive Web Application
Command to use: `ng add @angular/pwa`

# Angular Material
1. Generate a new module an add it to the list of imports in the main `AppModule`.
2. Keep all Material module imports in the separate module

```ts
import { MatButtonModule } from '@angular/material/button';
import { MatIconModule} from '@angular/material/button';
const MATERIAL = [ MatButtonModule, MatIconModule ];

@NgModule({
	imports: MATERIAL,
	exports: MATERIAL
})
export class MaterialModule { }
```

# Template reference
```html
<h1 #h1Element>hello, world</h1>

<!-- Assign the template ref on a <form> to ngForm object -->
<form #form="ngForm" (ngSubmit)="processForm(form)">
...
</form>
```
```ts
processForm(form: ngForm) {
	const name = form.value.name;
	...
	form.reset();
}
```
*Only use these with template-driven forms.*

```ts
form: FormGroup;

this.form.reset()

//Reset the form to specific values
this.form.reset({
	username: 'user',
	email: 'user@example.com'
})

//Reset individual controls
this.form.get('username').reset();
```

# Observable
Operators within pipe: 
1. filter -- filter data stream
2. map
3. tap -- observe the data stream. Used to perform side-effects
4. take -- take the first n values
5. takeWhile -- continue taking data until predicate is false
6. skip, skipWhile
7. switchMap -- change to a different stream

Subscribing
```ts
this.sub$ = this.form.valueChanges.subscribe({
	next: (data) => { console.info(data[0].name) },
	error: (err) => {console.error(err)},
	complete: () => {this.sub$.unsubscribe()}
})
```
Unsubscribing
```
ngOnDestroy() {
	this.valueChanges$.unsubscribe()
}
```

# Promise
Provider:
```ts
const callMe = new Promise((resolve, reject) => {
	resolve(data);
	reject(error);
}
```
Consumer:
```ts
callMe
	.then((data) => { })
	.catch((err) => { })
```

# HTTP Requests
```ts
this.sub$ = this.http.get<User[]>(url)
	.subscribe({
		next: (data) => { console.info(data[0].name) },
		error: (err HttpErrorResponse) => { console.error(err) },
		complete: () => {this.sub$.unsubscribe() }
	})
```
```ts
this.data$ = lastValueFrom(this.http.get<User[]>(url)
	.then((data) => {data.name})
	.catch((err: HttpErrorResponse) => { ... })
```

## With params and headers
```ts
queryParams = new HttpParams()
	.set("custId", 1234);
const headers = new HttpHeaders()
	.set('Authorization', `Bearer ${jwt}`);
	
this.http.get(url, body, {
	headers: headers,
	params: queryParams
});
```

## Async pipe
If data is a promise:
```html
<div *ngIf="data$ | async as u; else loading">
	<user-info [user]="u"></user-info>
</div>

<ng-template #loading>
	Loading...
</ng-template>
```
*Will wait for promise to resolve. In the meantime, will show loading.*

If data is an observable:
```html
<ul>
	<li *ngFor="let u of data$ | async>
		<user-info [user]="u"></user-info>
	</li>
</ul>
```

# Cross Origin
In the Controller:
```java
@RestController
@RequestMapping(...)
@CrossOrigin(origins="*")
public class DemoRestController {
	@GetMapping(...)
	@CrossOrigin(origins="*")
	public ...
}
```

## Enabling CORS globally
```java
public class EnableCORS implments WebMvcConfigurer {
	@Override
	public void addCorsMappings(CorsRegistry registry) {
		registry.addMapping("/api/*")
			.allowedOrigins("*");
	}
}
```

Add as bean (in SpringApplication)
```java
@Bean
public WebMvcConfigurer corsConfigurer() {
	return new EnableCORS();
}
```

## Proxying
1. Serve Angular app with `--proxy-config <file>` flag
2. Configure the application configuration by adding the following option:
	```json
	{
		"projects": {
			"<app_name>": {
				"architect": {
					"serve": {
						"builder": "@angular-devkit/build-angular:dev-server",
						"options": {
							"proxyConfig": "proxy.config.json"
						}
					}
				}
			}
		}
	}
	```

Proxy file:
```json
{
	"/api": {
		"target": "http://localhost:8080",
		"secure": false
	}
}
```

# Routing
Add import into module (auto-generated if routing is set to true during initialisation):
```ts
import { RouterModule } from '@angular/router';

const appRoutes: Routes = [
	{path: '', component: HomeComponent},
	{path: 'home/:id', component: HomeComponent},
	{path: '**', redirectTo: '/', pathMatch: 'full'}
]

@NgModule({
	imports: [
		...,
		RouterModule.forRoot({appRoutes})
	]
})
```

`router-outlet` is used to demarcate where in the app shell the component should be displayed. 

## Navigating routes
```html
<a [routerLink]="['/home']">Home</a>
<a [routerLink]="['/home', id]">Home</a>
<a [routerLink]="['/home', id]" [queryParams]="{view: 'simple'}">
	>Home</a>
```

```ts
this.router.navigate(['/home']);
this.router.navigate(['/home', id]);
this.router.navigate(['/home', id], {queryParams: {view: 'simple'});
```

## Accessing routes
```ts
this.activatedRoute.snapshot.params.id;
this.activatedRoute.snapshot.queryParams.view;
```
Another method is to use `@Input()`.
```ts
@Input() id: string

RouterModule.forRoot(appRoutes, {bindToComponentInputs: true});
```

# Route guards
## `CanActivate`
```ts
export const checkIfAuthenticated = (route: ActivatedRouteSnapshot, state: RouterStateSnapshot) => {
	const authSvc = inject(AuthService);
	const router = inject(Router);
	
	if (authSvc.isAuthenticated())
		return true;
	// Create a UrlTree to navigate to /help
	return router.parseUrl('/help')
}

{path: 'customers', component: CustomerListComponent, canActivate: [checkIfAuthenticated]}
```

## `CanDeactivate`
```ts
export const hasSaved: CanDeactivateFn<OrderFormComponent> = (orderForm: OrderFormComponent, 
	route: ActivatedRouteSnapshot, state: RouterStateSnapshot) => {
	
	if (orderForm.form.dirty)
		return confirm('You have not saved the order.\nAre you sure you want to leave?')

	return true;
}

{path:..., canDeactivate: [hasSaved]}
```

# File upload
mysql types: tinyblob, blob, mediumblob, longblob

## Template
Application properties in Spring Boot:
```properties
spring.servlet.multipart.enabled=true
spring.servlet.multipart.max-file-size=100MB
spring.servlet.multipart.max-request-size=200MB
spring.servlet.multipart.file-size-threshold=1MB
```

Server:
```java
@PostMapping(consumes=MediaType.MULTIPART_FORM_DATA_VALUE)
public ResponseEntity<String> postUpload(@RequestPart MultipartFile file, @RequestPart String name, @RequestPart String email) {
	String name = file.getName();
	String originalName = file.getOriginalFileName();
	String mediaType = file.getContentType();
	InputStream is = file.getInputStream();
}

template.update("insert into files(..., content) values (..., ?)", ..., is);
```
Angular:
```ts
export class FileUploadComponent {
	form!: FormGroup;
	dataUri!: string;
	blob!: Blob;
	isLoading: boolean = false;

	upload() {
		if (!this.dataUri) {
			console.error("No image uploaded");
			return;
		}
		
		// Move this to service class
		const formData = new FormData();
		formData.set('comments', this.form.get('comments')?.value);
		formData.set('picture', this.dataUriToBlob(this.dataUri));

		this.http.post(url, formData);
	}

	onImageChange(event: Event) {
		this.isLoading = true;
		const input = event.target as HTMLInputElement;
		if (input.files && input.files.length > 0) {
			const file = input.files[0];
			const reader = new FileReader();
			reader.onload = () => {
				this.dataUri = reader.result as string;
				this.isLoading = false;
			}
			reader.readAsDataURL(file);
			
		}
	}
}
```
*Setting the form control value as the `dataUri` does not work. Will be undefined*

```ts
dataUriToBlob(dataUri: string): Blob {
	const [meta, base64Data] = dataUri.split(',');
	const mimeMatch = meta.match(/:(.*?);/);

	const mimetype = mimeMatch ? mimeMatch[1] : 'application/octet-stream';
	const byteString = atob(base64Data);
	const ab = new ArrayBuffer(byteString.length);
	const ia = Uint8Array(ab);
	for (let i = 0; i < byteString.length; i++) {
		ia[i] = byteString.charCodeAt(i);
	}
	return new Blob([ia], {type: mimeType});
}
```

HTML page:
```html
<form enctype="multipart/form-data" (submit)="upload()">
	<input type="file" (change)="onImageChange($event)" accept="image/*">
</form> 
```
## Retrieving BLOB
Single row:
```java
Optional<FileData> opt = template.query("select * from files where id = ?", params, (ResultSet rs) -> {
	
	if (!rs.next())
		return Optional.empty();
	FileData file = new FileData();
	file.setName(rs.getString("name"));
	file.setContentType(rs.getString("media_type"));
	file.setContent(rs.getBytes("content"));

	return Optional.of(file);
}, id
)
```
Multiple rows:
```java
List<FileData> files = template.query("select * from files where name like ?", 
	params, (ResultSet rs) -> {
		List<FileData> list = new LinkedList<>();
		while (rs.next()) {
			FileData file = new FileData();
			file.setName(rs.getString("name"));
			file.setContentType(rs.getString("media_type"));
			file.setContent(rs.getBytes("content");
			list.add(file);
		}
		return list;
	}, "%dog%"
)
```
Sending over HTTP:
```java
byte[] image = ...;
String encodedString = Base64.getEncoder()
	encodeToString(image);
String payload = "data:image/png;base64," + encodedString;
```
```java
if (opt.isPresent()) {
	FileData file = opt.get();
	
	HttpHeaders headers = new HttpHeaders();
	headers.setContentType(MediaType.parseMediaType(file.getContentType()));

	// Can choose either inline display or attachment download. For images, we typically want inline display
	headers.setContentDispositionFormData("inline", file.getName());

return ResponseEntity.ok()
	.headers(headers)
	.body(file.getContent());
}
```

# Browser storage
## Local/session storage
```js
localstorage.setItem(key, value)
localstorage.getItem(key)

localStorage['key'] = 'value'
localStorage['key']

localStorage.removeItem('key')
localStorage['key'] = null // sets value to null, does not remove item

localStorage.clear()
```

## IndexedDB
Install:
```
npm i dexie
```

Database creation:
```ts
import Dexie from 'dexie';

export class MyStore extends Dexie {
	//Table with Cart as the schema, and the pk is number
	cart: Dexie.Table<Cart, number>;

	constructor() {
		super('MyStoreDB') // Database name
		this.version(1).stores({
			cart: '++cartId' // Collection name
		})
		//Hold a reference of the collection
		this.cart = this.table('cart')
	}
}
```

```ts
// Index multiple attributes (do not index blobs)
cart: '++cartId, username'
// Non-auto-incremented primary key
cart: 'username'
//Composite primary key
cart: '[cartId+username]'
```

### Methods
Return the entire collection as an array
```ts
const carts: Cart[] = await this.cart.toArray()
```
Return a limited amount of documents, with offset
```ts
const carts: Cart[] = this.cart
	.offset(50).limit(50)
	toArray()
```

Processing one document at a time
```ts
this.cart
	.orderBy('date').reverse()
	.each(c => {...})
```
Find document by primary key
```ts

```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3OTE5OTM1OTEsLTI4NTE1MTYwNiwxND
kzNzM3ODUzLC0xMzE3MDgwMjIsLTU4MTA3MjQ2Nl19
-->