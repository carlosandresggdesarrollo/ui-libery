<template>
	<fieldset class=" pkpFormField--metadata"> <!-- pkpFormField pkpFormField--options-->
		<legend class="pkpFormField--options__legend">
			<!--{{ label }}-->
			<tooltip v-if="tooltip" aria-hidden="true" :tooltip="tooltip" label="" />
			<span
				v-if="tooltip"
				class="-screenReader"
				:id="describedByTooltipId"
				v-html="tooltip"
			/>
			<help-button
				v-if="helpTopic"
				:id="describedByHelpId"
				:topic="helpTopic"
				:section="helpSection"
				:label="__('help.help')"
			/>
		</legend>
		<div
			v-if="description"
			class="pkpFormField__description parrafo_textopkpFormField--options__description"
			v-html="description"
			:id="describedByDescriptionId"
		/>
		<field-error
			v-if="errors && errors.length"
			:id="describedByErrorId"
			:messages="errors"
		/>
		<div class="pkpFormField__control">
			<label
				v-for="option in options"
				:key="option.value"
				class="pkpFormField--options__option"
			>
				<input
					class="form-check-input"
					type="checkbox"
					v-model="isEnabled"
					:value="option.value"
					:aria-describedby="describedByIds"
					:aria-invalid="errors && errors.length"
					:disabled="option.disabled"
					style="margin-right:30px;"
				/>
				<span
					class="pkpFormField--options__optionLabel"
					v-html="option.label"
				/>
			</label>
			<div v-if="isEnabled" class="pkpFormField--metadata__submissionOptions">
				<label
					v-for="option in submissionOptions"
					:key="'submission' + option.value"
					class="pkpFormField--options__option"
				>
					<input
						class="pkpFormField--options__input pkpFormField--metadata__submissionInput"
						v-model="selectedValue"
						type="radio"
						:value="option.value"
						:aria-invalid="errors && errors.length"
						:disabled="option.disabled"
						style="margin-right:30px;"
					/>
					<span
						class="pkpFormField--options__optionLabel"
						v-html="option.label"
					/>
				</label>
			</div>
		</div>
	</fieldset>
</template>

<script>
import FieldOptions from './FieldOptions.vue';

export default {
	name: 'FieldMetadataSetting',
	extends: FieldOptions,
	props: {
		// The value which matches a disabled state
		disabledValue: {
			type: Number,
			required: true
		},
		// The value which matches an enabled state, but not when it is requested
		// or required during submission.
		enabledOnlyValue: {
			type: String,
			required: true
		},
		value: {
			type: [Number, String],
			required: true
		},
		submissionOptions: {
			type: Array,
			required: true
		}
	},
	data() {
		return {
			isEnabled: this.disabledValue !== this.value
		};
	},
	watch: {
		/**
		 * Whenever the submission value changes, emit an event to update the value
		 * of this field in the form component.
		 */
		isEnabled: function(newVal, oldVal) {
			if (newVal === oldVal) {
				return;
			}
			this.selectedValue = newVal ? this.enabledOnlyValue : this.disabledValue;
		}
	}
};
</script>

<style lang="less">
@import '../../../styles/_import';

.pkpFormField--metadata__submissionOptions {
	margin: 1rem -1rem 0;
	padding: 1rem 1rem 0;
	border-top: @bg-border;
}
</style>
