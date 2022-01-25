# VueDatePicker
A DatePicker built with Vue 3, Bootstrap 5, and TypeScript

This Vue component can be dropped right in and used by simply importing it as you would any component, then using it like this: 

```html
<DatePickerVue @selected-date="startDate = $event" :date="startDate" />
```

In this case, I am setting the date to a variable `startDate`. When the date is chosen, it is passed up in the form of an event, setting `startDate` to the newly chosen date. That updated date is then passed back to the datepicker.
