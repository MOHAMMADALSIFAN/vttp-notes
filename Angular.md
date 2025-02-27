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
| property | purpose|
---------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzNTU2OTY0MzIsNTA0ODE1NjY0LC0xMT
c4MTUzMDY0XX0=
-->