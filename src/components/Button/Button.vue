<template>
	<component
		:is="element"
		class="btn btn-primary"
		:class="classes"
		:href="element === 'a' ? href : false"
		:disabled="isDisabled"
		@click="click"
		@focus="$emit('focus')"
		@blur="$emit('blur')"
	>
		<slot />
	</component>
</template>

<script>
export default {
	name: 'pkpButton',
	props: {
		element: {
			type: String,
			default: 'button'
		},
		href: String,
		isPrimary: Boolean,
		isWarnable: Boolean,
		isActive: Boolean,
		isLink: Boolean,
		isDisabled: Boolean
	},
	computed: {
		classes() {
			let classes = [];
			if (this.isPrimary) {
				classes.push('btn btn-primary');
			}
			if (this.isWarnable) {
				classes.push('btn btn-primary');
			}
			if (this.isActive) {
				classes.push('btn btn-primary');
			}
			if (this.isLink) {
				classes.push('btn btn-primary');
			}
			return classes;
		}
	},
	methods: {
		click(e) {
			if (this.element === 'button') {
				e.preventDefault();
			}
			this.$emit('click');
		}
	}
};
</script>
