<template>
	<form class="" :method="method" :action="action">
		<input
			v-for="(value, name) in hiddenFields"
			:key="name"
			type="hidden"
			:name="name"
			:value="value"
		/>
		<form-locales
			v-if="availableLocales.length > 1"
			:primaryLocaleKey="primaryLocale"
			:locales="availableLocales"
			:visible="visibleLocales"
			@updateLocales="setVisibleLocales"
		/>
		<div v-if="pages.length > 1" class="pkpForm__pageNav">
			<ol class="pkpForm__pageNavList">
				<li
					v-for="page in pages"
					:key="page.id"
					class="pkpForm__pageNavListItem"
				>
					<button
						class="btn btn-primary"
						:class="{'pkpForm__pageNavPage--current': page.id === currentPage}"
						@click.prevent="setCurrentPage(page.id)"
					>
						{{ page.label }}
					</button>
					<icon
						v-if="pageIdsWithErrors.includes(page.id)"
						icon="exclamation-triangle"
						:inline="true"
					/>
				</li>
			</ol>
		</div>
		<div class="pkpFormPages" :class="classes">
			<form-page
				v-for="(page, index) in pages"
				:key="page.id"
				v-bind="page"
				:groups="groups"
				:fields="fields"
				:errors="errors"
				:formId="id"
				:canSubmit="canSubmit"
				:isCurrentPage="currentPage === page.id"
				:isLastPage="index === pages.length - 1"
				:lastSaveTimestamp="lastSaveTimestamp"
				:primaryLocale="primaryLocale"
				:visibleLocales="visibleLocales"
				:availableLocales="availableLocales"
				:isSaving="isSaving"
				@change="fieldChanged"
				@pageSubmitted="nextPage"
				@previousPage="setCurrentPage(false)"
				@showField="showField"
				@showLocale="showLocale"
				@set-errors="setErrors"
			/>
		</div>
	</form>
</template>

<script>
import FormLocales from './FormLocales.vue';
import FormPage from './FormPage.vue';

export default {
	name: 'PkpForm',
	components: {
		FormLocales,
		FormPage
	},
	props: {
		id: String,
		method: {
			type: String,
			default() {
				return '';
			}
		},
		action: {
			type: String,
			default() {
				return '';
			}
		},
		canSubmit: {
			type: Boolean,
			default() {
				return true;
			}
		},
		errors: {
			type: Object,
			default() {
				return {};
			}
		},
		fields: Array,
		groups: Array,
		hiddenFields: Object,
		pages: Array,
		primaryLocale: String,
		visibleLocales: Array,
		supportedFormLocales: Array
	},
	data() {
		return {
			currentPage: '',
			isSaving: false,
			lastSaveTimestamp: -1
		};
	},
	computed: {
		/**
		 * Classes to add to wrapper element
		 *
		 * @return {Array}
		 */
		classes() {
			let classes = [];
			if (this.visibleLocales.length > 1) {
				classes.push('pkpForm--hasManyVisibleLocales');
			}
			return classes;
		},

		/**
		 * Which locales should be available in this form?
		 *
		 * @return {Array}
		 */
		availableLocales() {
			return this.hasMultilingualFields ? this.supportedFormLocales : [];
		},

		/**
		 * Has multilingual fields
		 *
		 * @return {Boolean}
		 */
		hasMultilingualFields() {
			return !!this.fields.find(field => field.isMultilingual);
		},

		/**
		 * List of group ids which contain a field with an error
		 *
		 * @return {Array}
		 */
		groupIdsWithErrors() {
			let groupIds = [];
			Object.keys(this.errors).forEach(fieldName => {
				const field = this.fields.find(field => field.name === fieldName);
				if (field && !groupIds.includes(field.groupId)) {
					groupIds.push(field.groupId);
				}
			});
			return groupIds;
		},

		/**
		 * List of page ids which contain a field with an error
		 *
		 * @return {Array}
		 */
		pageIdsWithErrors() {
			let pageIds = [];
			this.groupIdsWithErrors.forEach(groupId => {
				const group = this.groups.find(group => group.id === groupId);
				if (group && !pageIds.includes(group.pageId)) {
					pageIds.push(group.pageId);
				}
			});
			return pageIds;
		},

		/**
		 * The values to send when submitting the form
		 *
		 * Strips out values from locales that are not currently in the form's
		 * this.supportedFormLocales array. If the locales have changed and old
		 * data is still around, it can cause validation errors when trying to
		 * submit to the API.
		 *
		 * @return {Object}
		 */
		submitValues() {
			let values = {};
			this.fields.forEach(field => {
				if (field.component === 'field-html') {
					return;
				}
				if (!field.isMultilingual) {
					// Convert empty arrays to an empty string to address the fact that
					// jQuery drops empty arrays before sending the POST data.
					// See: https://bugs.jquery.com/ticket/6481
					if (Array.isArray(field.value) && !field.value.length) {
						values[field.name] = '';
					} else {
						values[field.name] = field.value;
					}
				} else {
					let fieldValues = {};
					Object.keys(field.value).forEach(localeKey => {
						if (
							this.supportedFormLocales.filter(
								locale => locale.key === localeKey
							).length
						) {
							// See note above on converting empty arrays
							if (
								Array.isArray(field.value[localeKey]) &&
								!field.value[localeKey].length
							) {
								fieldValues[localeKey] = '';
							} else {
								fieldValues[localeKey] = field.value[localeKey];
							}
						}
					});
					if (Object.keys(fieldValues).length) {
						values[field.name] = fieldValues;
					}
				}
			});
			return {
				...values,
				...this.hiddenFields
			};
		}
	},
	methods: {
		/**
		 * Move to the next page or submit form if we're at the last page
		 */
		nextPage: function(pageId) {
			const pageIndex = this.pages.findIndex(page => page.id === pageId);
			if (this.pages.length === 1 || pageIndex === this.pages.length - 1) {
				this.submit();
				return;
			}
			this.setCurrentPage(true);
		},

		/**
		 * Submit the form
		 */
		submit() {
			if (!this.canSubmit) {
				return false;
			}

			this.isSaving = true;

			let errors = this.validate();
			if (Object.keys(errors).length) {
				this.$emit('set', this.id, {
					errors: {
						...this.errors,
						...errors
					}
				});
				this.isSaving = false;
				return;
			}

			if (this.action === 'emit') {
				this.$emit('success', this.submitValues);
			} else {
				$.ajax({
					context: this,
					method:
						this.method === 'DELETE' || this.method === 'PUT'
							? 'POST'
							: this.method,
					url: this.action,
					headers: {
						'X-Csrf-Token': pkp.currentUser.csrfToken,
						'X-Http-Method-Override': this.method
					},
					data: this.submitValues,
					success: this.success,
					error: this.error,
					complete: this.complete
				});
			}
		},

		/**
		 * Validate the form
		 *
		 * @return {Object}
		 */
		validate() {
			return this.validateRequired();
		},

		/**
		 * Check if required form fields are present
		 *
		 * @return {Object}
		 */
		validateRequired() {
			let errors = {};
			this.fields.forEach(field => {
				if (!field.isRequired) {
					return;
				}
				let missingValue = false;
				// Only require the primary locale by default for multilingual fields
				let value = field.isMultilingual
					? field.value[this.primaryLocale]
					: field.value;
				switch (typeof value) {
					case 'undefined':
						missingValue = true;
						break;
					case 'string':
					case 'array':
						if (!value.length) {
							missingValue = true;
						}
						break;
					case 'object':
						// null values are stored as objects
						if (!value) {
							missingValue = true;
						}
						break;
				}
				if (missingValue) {
					if (field.isMultilingual) {
						errors[field.name] = {};
						errors[field.name][this.primaryLocale] = [
							this.__('validator.required')
						];
					} else {
						errors[field.name] = [this.__('validator.required')];
					}
				}
			});
			return errors;
		},

		/**
		 * Callback to fire when the form submission's ajax request has been
		 * returned successfully
		 *
		 * @param {Object} r The response to the AJAX request or the values
		 * 		of the form if no request was sent
		 */
		success: function(r) {
			this.$emit('success', r);
			this.lastSaveTimestamp = Date.now();
			pkp.eventBus.$emit('form-success', this.id, r);

			if (this.action) {
				// Update form values with the response values
				const newFields = this.fields.map(field => {
					if (typeof r[field.name] !== 'undefined') {
						field.value = r[field.name];
					}
					return field;
				});
				this.$emit('set', this.id, {fields: newFields});
			}
		},

		/**
		 * Callback to fire when the form submission's ajax request has been
		 * returned with errors
		 *
		 * @param {Object} r The response to the AJAX request
		 */
		error: function(r) {
			// Field validation errors
			if (r.status && r.status === 400) {
				pkp.eventBus.$emit(
					'notify',
					this.__('form.errors', {
						count: Object.keys(r.responseJSON).length
					}),
					'warning'
				);
				this.$emit('set', this.id, {errors: r.responseJSON});
				// A generic error from the API endpoint
			} else if (
				r.status &&
				[403, 404].includes(r.status) &&
				r.responseJSON &&
				r.responseJSON.errorMessage
			) {
				pkp.eventBus.$emit('notify', r.responseJSON.errorMessage, 'warning');
			} else {
				pkp.eventBus.$emit('notify', this.__('common.unknownError', 'warning'));
			}
		},

		/**
		 * Callback to fire when the form submission's ajax request has been
		 * returned, and the success or error callbacks have already been fired.
		 *
		 * @param {Object} r The response to the AJAX request
		 */
		complete() {
			this.isSaving = false;
		},

		/**
		 * Update values when a field has changed
		 *
		 * @param {String} name Name of the field to modify
		 * @param {String} prop Name of the prop to modify
		 * @param {mixed} value The new value for the prop
		 * @param {String} localeKey Optional locale key for multilingual props
		 */
		fieldChanged: function(name, prop, value, localeKey) {
			const newFields = this.fields.map(field => {
				if (field.name === name) {
					if (localeKey) {
						field[prop][localeKey] = value;
					} else {
						field[prop] = value;
					}
				}
				return field;
			});
			this.$emit('set', this.id, {fields: newFields});
			this.removeError(name, localeKey);
		},

		/**
		 * Set the current page
		 *
		 * @param {Boolean|Number} pageId The id of the page you want to display.
		 *  You can also pass `true` to move to the next page and `false` to move to
		 *  the previous page.
		 */
		setCurrentPage: function(pageId) {
			const currentPageIndex = this.pages.findIndex(
				page => page.id === this.currentPage
			);
			if (pageId === true) {
				if (this.pages.length <= currentPageIndex) {
					return;
				}
				pageId = this.pages[currentPageIndex + 1].id;
			} else if (pageId === false) {
				if (!currentPageIndex) {
					return;
				}
				pageId = this.pages[currentPageIndex - 1].id;
			}
			this.currentPage = pageId;
			this.$scrollTo(this.$el, 500, {
				offset: -50
			});
		},

		/**
		 * Update visible locales
		 *
		 * @param {Array} locales New array of visible locales
		 */
		setVisibleLocales: function(locales) {
			this.$emit('set', this.id, {visibleLocales: locales});
		},

		/**
		 * Display a particular locale
		 *
		 * @param {String} localeKey
		 */
		showLocale: function(localeKey) {
			if (this.visibleLocales.includes(localeKey)) {
				return;
			}
			this.setVisibleLocales([this.primaryLocale, localeKey]);
		},

		/**
		 * Display and scroll to a field in the form
		 *
		 * @param {String} name The name of the field to bring into view
		 */
		showField: function(name) {
			let field = this.fields.find(field => field.name === name);
			if (!field) {
				return;
			}
			let group = this.groups.find(group => group.id === field.groupId);
			if (!group) {
				return;
			}
			this.currentPage = group.pageId;
			// If the page has changed, we need to wait for it to be rendered in
			// $nextTick before we can find and scroll to it
			this.$nextTick(() => {
				// Reach directly into the dom to locate the field so we can scroll to it
				// This pattern may match several elements related to the field, any one
				// of which will be ok to scroll to.
				let $el = document.querySelector(
					'[id*="' + this.id + '-' + field.name + '"]'
				);
				if ($el) {
					this.$scrollTo($el, 500, {
						offset: -50
					});
				} else {
					this.setCurrentPage(group.pageId);
				}
			});
		},

		/**
		 * Remove an error from the error list
		 *
		 * @param {String} name The name of the field
		 * @param {String|Null} localeKey The locale to remove, if not removing the
		 *  whole error.
		 */
		removeError: function(name, localeKey) {
			if (!this.errors[name]) {
				return;
			}
			// Vue.js can't detect changes in subproperties, so replace the whole
			// object to trigger a re-render of the effected templates
			let newErrors = {...this.errors};
			if (!localeKey || typeof newErrors[name] === 'string') {
				delete newErrors[name];
			} else {
				if (newErrors[name][localeKey]) {
					delete newErrors[name][localeKey];
				}
				if (!Object.keys(newErrors[name]).length) {
					delete newErrors[name];
				}
			}
			this.$emit('set', this.id, {errors: newErrors});
		},

		/**
		 * Emit an event to set a new errors array
		 *
		 * @param {Array} errors The new list of errors
		 */
		setErrors: function(errors) {
			this.$emit('set', this.id, {errors: errors});
		}
	},
	mounted() {
		// Set the current page
		if (!this.currentPage) {
			this.currentPage = this.pages[0].id;
		}
	}
};
</script>

<style lang="less">
@import '../../styles/_import';

.pkpForm {
	position: relative;
}

.pkpForm__pageNav {
	float: left;
	padding-right: 1rem;
	width: 20%;

	+ .pkpFormPages {
		margin-left: 20%;
	}

	&:after {
		position: absolute;
		content: '';
		top: 0;
		bottom: 0;
		left: 20%;
		border-left: @bg-border-light;
	}
}

.pkpForm__pageNavList {
	margin: 0;
	padding: 1rem;
	list-style: none;
}

.pkpForm__pageNavListItem {
	position: relative;
	font-size: @font-sml;
	line-height: 1.5em;

	+ .pkpForm__pageNavListItem {
		margin-top: 1em;
	}

	.fa {
		position: absolute;
		top: 50%;
		left: 100%;
		transform: translate(50%, -50%);
	}
}

.pkpForm__pageNavPage {
	display: inline-block;
	margin: 0;
	padding: 0;
	border: none;
	background: transparent;
	box-shadow: none;
	text-align: left;
	color: @primary;
	cursor: pointer;
}

.pkpForm__pageNavPage--current {
	font-weight: @bold;
}

.pkpFormPage {
	position: absolute;
	left: -9999px;
}

.pkpFormPage--current {
	position: relative;
	left: auto;
}
</style>
