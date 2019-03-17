<template>
  <div v-if="shown">
    <div class="modal fade" ref="modalEl">
      <div class="modal-dialog" :class="modalClass">
        <div class="modal-content">
          <div class="modal-header">
            <slot name="header"></slot>
          </div>
          <div class="modal-body">
            <slot></slot>
          </div>
          <div class="modal-footer">
            <slot name="footer">
              <div class="text-right">
                  <button type="button" class="btn btn-light border mr-3" @click="cancel()">{{ cancelLabel }}</button>
                  <button type="button" class="btn btn-primary border" @click="_process()" :disabled="busy">{{ actionLabel }}</button>
              </div>
            </slot>
          </div>
        </div>
      </div>
    </div>        
  </div>
</template>
<script>
export default {

  props: {
    isLarge: {type: Boolean, default: false},
    cancelLabel: {type: String, default: 'Cancel'},
    actionLabel: {type: String, default: 'OK'},
    handler: {type: Function, default: () => Promise.resolve(true)},
  },

  data() {
    return {
      busy: false,
      shown: false,
      onSuccess: null,
      onCancel: null,
      modalClass: this.isLarge ? 'modal-lg' : 'modal-default',
    };
  },

  methods: {

    /**
     * Shows the modal.
     * Returns a promise that resolves when the modal is shown.
     */
    show() {
      this.shown = true;

      return new Promise((resolve) => {
        Vue.nextTick(() => {
          $(this.$refs.modalEl).one('show.bs.modal', () => resolve());
          $(this.$refs.modalEl).one('hide.bs.modal', () => this._onModalClosed());
          $(this.$refs.modalEl).modal('show');
        });
      });
    },

    /**
     * Cancel the modal prematurely. Closes the modal.
     * If waitForResponse was called, it will be resolved with a rejection.
     */
    cancel() {
        this.busy = false;
        this._closeModal();
    },    

    /**
     * Waits until the action in the modal is finished or cancelled.
     * Pressing the action button will call the "handler" prop that you passed.
     * it will wait in busy state until the promise is resolved, then return its value.
     * If cancel is pressed or the modal is closed by pressing escape, you will get a rejection.
     */
    waitForResponse() {
      return new Promise((resolve, reject) => {
          this.onCancel = reject
          this.onSuccess = resolve
      });
    },

    _onModalClosed() {
      this.shown = false;
      this._failPromise('Cancelled');
    },

    _finishPromise(data) {
        if(this.onSuccess) {
            this.onSuccess(data)
            this._clearHandlers()
        }
    },
    
    _failPromise(msg) {
        if (this.onCancel) {
            this.onCancel(new Error(msg))
            this._clearHandlers()
        }
    },
    
    _clearHandlers() {
        this.onCancel = null
        this.onSuccess = null  
    },    

    _closeModal() {
        $(this.$refs.modalEl).modal('hide');
    },

    _process() {
        this.busy = true
        this.handler().then((response) => {
            this.busy = false
            this._finishPromise(response)
            this._closeModal()
        }).catch((error) => {
            this.busy = false
            // ERROR...
        })
    }
  },
}
</script>