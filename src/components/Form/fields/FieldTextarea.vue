<template>
	<div class="pkpFormField pkpFormField--textarea" :class="classes">
		<div class="pkpFormField__heading">
			 <form-field-label
			 	class="título_nivel_2"
				:controlId="controlId"
				:label="label"
				:localeLabel="localeLabel"
				:isRequired="isRequired"
				:requiredLabel="__('common.required')"
				:multilingualLabel="multilingualLabel"
			/>
			<tooltip
				v-if="isPrimaryLocale && tooltip"
				aria-hidden="true"
				:tooltip="tooltip"
				label=""
			/>
			<span
				v-if="isPrimaryLocale && tooltip"
				class="-screenReader"
				:id="describedByTooltipId"
				v-html="tooltip"
			/>
			<help-button
				v-if="isPrimaryLocale && helpTopic"
				:id="describedByHelpId"
				:topic="helpTopic"
				:section="helpSection"
				:label="__('help.help')"
			/>
		</div>
		<div
			v-if="isPrimaryLocale && description"
			class="pkpFormField__description"
			v-html="description"
			:id="describedByDescriptionId"
		/>
		<div class="pkpFormField__control">
			<textarea
				class="form-control"
				v-model="currentValue"
				:id="controlId"
				:name="localizedName"
				:aria-describedby="describedByIds"
				:aria-invalid="errors && errors.length"
				:required="isRequired"
			></textarea>
			<multilingual-progress
				v-if="isMultilingual && locales.length > 1"
				:id="multilingualProgressId"
				:count="multilingualFieldsCompleted"
				:total="locales.length"
			/>
		</div>
		<field-error
			v-if="errors && errors.length"
			:id="describedByErrorId"
			:messages="errors"
		/>
	</div>
</template>

<script>
import FieldBase from './FieldBase.vue';

export default {
	name: 'FieldTextarea',
	extends: FieldBase,
	props: {
		size: {
			type: String,
			default: 'normal',
			validator: function(value) {
				return ['small', 'normal', 'large'].indexOf(value) !== -1;
			}
		}
	},
	computed: {
		/**
		 * Classes added to the wrapper element
		 *
		 * @return {Array}
		 */
		classes() {
			return ['pkpFormField--' + this.size];
		}
	}
};
</script>

<style lang="less">
@import '../../../styles/_import';
.título_nivel_2 {
	font-family:cambersb;
	font-size:18pt;
	color:#737373;
}
.pkpFormField--textarea__input {
	padding-top: 0.5em;
	padding-bottom: 0.5em;
	height: 20em;
	width: 100%;
	line-height: 1.8em;
}

.pkpFormField--small .pkpFormField--textarea__input {
	max-width: 20em;
	height: 8em;
}

.pkpFormField--large .pkpFormField--textarea__input {
	height: 40em;
}
</style>
