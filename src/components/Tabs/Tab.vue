<template>
	<div
		class=""
		:class="classes"
		role="tabpanel"
		:id="id"
		:aria-labelledby="id + '-button'"
		:hidden="!isActive"
	>
		<slot />
	</div>
</template>

<script>
export default {
	props: {
		icon: {
			type: String,
			default() {
				return '';
			}
		},
		id: {
			type: String,
			required: true
		},
		label: {
			type: String,
			default() {
				return '';
			}
		},
		badge: {
			type: [String, Number],
			default() {
				return '';
			}
		}
	},
	data() {
		return {
			isActive: false
		};
	},
	computed: {
		/**
		 * Classes to apply to root element
		 *
		 * @return {Array}
		 */
		classes() {
			let classes = [];
			if (this.isActive) {
				classes.push('');
			}
			if (this.isDisabled) {
				classes.push('');
			}
			return classes;
		}
	}
};
</script>
