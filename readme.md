Vue Bootstrap 4 Modal
=====================================

This is an opinionated bootstrap 4 modal component with promise support. Use this if you want to quickly open a Bootstrap 4 modal, implement a custom handler for the primary action button or listen to show / hide events.


### Add to Your Project

Add this package as a dependency:

```bash
yarn add vue-bs4-modal
```

Or you can just copy [src/Bs4Modal.vue](https://raw.githubusercontent.com/swiftmade/vue-bs4-modal/master/src/Bs4Modal.vue) into your project... Then do this:

```js
Vue.component('bs4-modal', require('vue-bs4-modal'));
```


#### Caveats 

* Make sure jQuery is globally accessible via `$`.
* You must include Bootstrap 4 script and stylesheet in your project.

### Properties

| Prop        | Type                | Description                                                                                                                                                         |
|-------------|---------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| handler     | Function -> Promise | The handler will be called when the action button is pressed. You MUST return a promise from your handler. UI will be in busy state until your promise is resolved. |
| actionLabel | String              | Label of your action button. By default, it will read "OK"                                                                                                          |
| cancelLabel | String              | Label of your cancel button. By default, it will read "Cancel"                                                                                                      |
| isLarge     | Boolean             | Pass true to make your modal large (`modal-lg`). Otherwise, `modal-default` will be used.                                                                           |

### Example Usage

In the following example, pressing "Open Modal" will open a bootstrap 4 modal. Pressing "OK" will put the modal in busy state for 5 seconds. Then the modal will close and `waitForResponse()` call will be resolved. If you cancel or close the modal without pressing OK, `waitForResponse()` call will receive a rejection.

```vue
<template>
  <div>
    <bs4-modal ref="modal" :handler="modalHandler">
      <div slot="header">Your Modal's Title</div>
      <div>Your Modal's body! Pressing OK will make you wait 5 seconds...</div>
    </bs4-modal>

    <button @click="openModal()">Open Modal</button>
  </div>
</template>

<script>
export default {
  methods: {
    
    openModal() {
      this.$refs.modal.show()
        .then(() => {
          console.log('Modal is shown!');
          return this.$refs.modal.waitForResponse();
        })
        .then(
          () => {
            console.log('Modal action is completed!');
          },
          () => {
            console.log('Modal is cancelled !');
          }
        );
    },
    
    modalHandler() {
      return new Promise(resolve => {
        setTimeout(resolve, 5000);
      });
    },
  }
}
</script>
```


