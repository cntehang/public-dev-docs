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
