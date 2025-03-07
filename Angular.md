# Foundation
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
2. Value binding `[attribute}="property"`

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
```ts
<div *ngIf="cart.length > 0"; else emptyCart>
	Your cart has {{cart.length}} item(s).
</div>

<ng-template #emptyCart> <!-- will be displayed
	Your cart is empty
</ng-template>
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwOTAxNjUwNTAsLTQwMDE5NTExMSwtND
g2MDM0MjksNTA0ODE1NjY0LC0xMTc4MTUzMDY0XX0=
-->