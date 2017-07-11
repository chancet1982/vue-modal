<template >
<div :class="['modal', open ? 'open' : 'closed']">
	<button class="modal-toggler modal-button" type="button" name="button" @click="openModal" v-show="showToggler"><slot name="modal-toggler">toggle modal</slot></button>
	<div v-if="overlay" class="modal-overlay"></div>
	<div  class="modal-dialog" :class="[{'has-header': hasHeader},{'has-footer': hasFooter}]" v-click-outside="closeIfOpen">
		<header class="modal-header" v-if="hasHeader">
		  <slot name="modal-header"></slot>
		  <i class="modal-header-close" @click="closeModal"><slot name="modal-header-icon"><svg viewBox="0 0 16 16" xmlns="http://www.w3.org/2000/svg" fill-rule="evenodd" clip-rule="evenodd" stroke-linejoin="round" stroke-miterlimit="1.414"><path d="M3.514 12.486c-.37-.37-.37-.97 0-1.34L6.66 8 3.513 4.855c-.37-.37-.37-.97 0-1.34l.01-.01c.365-.365.957-.365 1.322 0L8 6.66l3.145-3.146c.37-.37.97-.37 1.34 0l.01.01c.365.365.365.957 0 1.322L9.34 8l3.155 3.154c.365.365.365.957 0 1.322l-.01.01c-.37.37-.97.37-1.34 0L8 9.34l-3.145 3.146c-.37.37-.97.37-1.34 0z" fill="#374355"/></svg></slot></i>
		</header>
		<main class="modal-main">
			<div class="modal-main-padding">
		  		<slot>Place some content in me...</slot>
	  		</div>
		</main>
		<footer class="modal-footer" v-if="hasFooter">
		  <slot name="modal-footer"></slot>
		</footer>
	</div>
</div>
</template>

<script>
export default {
  name: 'modal',
	props: {
		isExternallyControlled: {
			required: false,
			type: Boolean,
			default: false,
		},
		isOpen: {
			required: false,
			type: Boolean,
			default: false,
		},
		overlay: {
			required: false,
			type: Boolean,
			default: true,
		},
		outsideClickClose: {
  		  type: Boolean,
  		  required: false,
  		  default: true,
  	  },
	},
	directives: {
      'click-outside': {
  		bind (el, binding, vnode) {
  		  el.event = (event) => {
  			if (!(el == event.target || el.contains(event.target) )) {
  			  vnode.context[binding.expression](event);
  			}
  		  };
  		  document.body.addEventListener('click', el.event)
  		},
  		unbind (el) {
  		  document.body.removeEventListener('click', el.event)
  		},
      },
    },
	computed :{
		hasHeader() {
	      return !!this.$slots['modal-header'];
	  	},
	    hasFooter() {
	      return !!this.$slots['modal-footer'];
	  	},
		showToggler() {
			return !this.isExternallyControlled;
		},
		open() {
			return this.isExternallyControlled ? this.isOpen : this.modalOpen;
		},
	},
	data() {
		return {
			modalOpen: this.isOpen,
		}
	},
	methods: {
		closeIfOpen(){
			//TODO doesnt work right now as first time when was clicked-outside might have been to open it.
		},
		openModal(){
			this.$emit("modal-open");
			this.toggleModal();
		},
		closeModal(){
			this.$emit("modal-close");
			this.toggleModal();
		},
		toggleModal() {
			if (!this.isExternallyControlled) {
				this.modalOpen = !this.modalOpen;
			}
			this.emitModalState();
		},
		emitModalState(){
			const payload = {
				open: this.open
			};
			this.$emit("modal-updated", payload);
		}
	},
}
</script>

<style lang="scss" scoped>
$main-padding: 20px;
$modal-header: 64px;
$modal-footer: 50px;
$close-icon-size: 16px;
$border-radius: 5px;
$modal-bg: #fff;
$modal-header-bg: darken($modal-bg,5%);
$modal-footer-bg: darken($modal-bg,5%);
$modal-border-color: darken($modal-bg,10%);

.modal {
	&.closed {
		> .modal-overlay {
			background-color: rgba(0,0,0,0);
			transform: scale(0,0);

		}
		> .modal-dialog {
			transform: translate(-50%, -30%) scale(0,0);
		}
	}
	&.open {
		> .modal-overlay {
			background-color: rgba(0,0,0,0.2);
			transform: scale(1,1);
		}
		> .modal-dialog {
			transform: translate(-50%, -50%) scale(1,1);
		}
	}
}
.modal-overlay {
	position: absolute;
	top: 0;
	right: 0;
	bottom: 0;
	left: 0;
	-webkit-transition: background-color .3s ease;
	transition: background-color .3s ease;
	overflow: hidden;
}

.modal-dialog {
	position:absolute;
	top:50%;
	left: 50%;
	transform: translate( -50%, -50%);
	box-shadow: 0 4px 8px rgba(0,0,0,0.4);
	min-width: 30%;
	min-height: 30%;
	height: 70vh;
	width: 70vw;
	background-color: $modal-bg;
	border-radius: $border-radius $border-radius $border-radius $border-radius;
	-webkit-transition: transform .4s ease .3s ;
	transition: transform .4s ease .3s ;
	transform-origin: 50% 50%;

	> .modal-header {
		position: relative;
		background-color: $modal-header-bg;
		border-bottom: solid 1px $modal-border-color;
		height:$modal-header;
		line-height: $modal-header;
		overflow: hidden;
		border-radius: $border-radius $border-radius 0 0;
		padding-right: $modal-header;
		padding-left: $main-padding;
		i {
			position: absolute;
			top:0;
			right: 0;
			width: $modal-header;
			height: $modal-header;
			cursor: pointer;
			svg {
				width: $close-icon-size;
				height: $close-icon-size;
				position: absolute;
				top:50%;
				left: 50%;
				transform: translate(-50%,-50%);
			}
		}
	}

	> .modal-main {
		height: auto;
		max-height: calc(100% - (#{$main-padding} * 2));
		overflow: auto;
		overflow-x: hidden;
		> .modal-main-padding {
			padding: $main-padding;
		}

	}

	&.has-header {
		> .modal-main {
			max-height: calc(100% - (#{$modal-header} + 1px));
		}
	}
	&.has-footer {
		> .modal-main {
			max-height: calc(100% - (#{$modal-footer} + 1px));
		}
	}
	&.has-header.has-footer {
		> .modal-main {
			max-height: calc(100% - (#{$modal-header} + #{$modal-footer} + 2px));
		}
	}

	> .modal-footer {
		background-color: $modal-footer-bg;
		border-top: solid 1px $modal-border-color;
		position: absolute;
		bottom: 0;
		left: 0;
		right: 0;
		border-radius:0 0 $border-radius $border-radius;
		height: $modal-footer;
		line-height: $modal-footer;
		overflow: hidden;
	}
}
</style>
