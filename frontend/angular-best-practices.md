# Angular Best Practices

Here is a set of best practices using Angular

## Forms

As hinted by the offical Angular document, use the new Reactive Form, not Template Form. Template forms are asynchornous and event-driven. They are also not as flexible as reactive forms. Reactive fomrs have the following benefits:

- Synchronous form controls are easier to unit test
- Forms and models are closely aligned
- Group form controls together at different levels using group and array.
- Validate form controls at any level and dynamically

Use the following style to create a form:

```ts
constructor(private formBuilder: FormBuilder) {}

ngOnInit() {
  this.createForm()
}

createForm() {
  this.myForm = this.formBuilder.group(...)
}
```

For most simple scenarios, use `FormBuilder` to create a form. For a complicated form, create `FormGroup` and `FormControl` directly.

## Dependency Injection

- Use `providedIn: 'root'` for services which should be available in whole application as singletons.
- Never use `providedIn: EagerlyImportedModule`, you don’t need it and if there is some super exceptional use case then go with the `providers: []` instead.
- Use `providedIn: LazyServiceModule` to prevent service injection in the eagerly imported part of the application. Use `LazyServiceModule` which will be imported by `LazyModule` to prevent circular dependency warning. LazyModule will then be lazy loaded using Angular Router for some route in a standard fashion. This is not recommended for too much confusing code.
- Use `providers: []` inside of `@Component` or `@Directive` to scope service only for the particular component sub-tree which will also lead to creation of multiple service instances (one service instance per one component usage)
- Always try to scope your services conservatively to prevent dependency creep and resulting tangled hell
