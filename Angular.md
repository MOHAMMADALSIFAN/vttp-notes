# Angular
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


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTQ4NjAzNDI5LDUwNDgxNTY2NCwtMTE3OD
E1MzA2NF19
-->